---
layout: post
title: SQL Server Temporal Tables and Rowversions
tags: sqlserver rowversions temporal tables
---

I was intrigued to see how a table with System_Versioning enabled would behave if it had a rowversion on the primary table, and how it would get inserted to the history table. 

The short answer is that it copies the rowversion of the previous change to the record into the history table. This feels slightly counterintuitive from a rowversion perspective as every change to a row will generate a new database level rowversion and set it on that record. But it is in-keeping with how temporal tables work in SQLServer (i.e. theres no state except the start/end dates so a record is copied exactly into the history table).

It should also be noted that there doesn't appear to be a way to add a rowversion to the history table either. I.e. it must match the primary tables columns, so no you can't just set the column to varbinary and add a new rowversion column that behaves more like a rowversion.

## Heres some Proof


    {% highlight SQL %}

        CREATE TABLE tmp (
            Id INT IDENTITY PRIMARY KEY NOT NULL,
            Name NVARCHAR(50),
            [RowVersion] TIMESTAMP,
            [SysStartTime] [DATETIME2](0) GENERATED ALWAYS AS ROW START NOT NULL,
            [SysEndTime] [DATETIME2](0) GENERATED ALWAYS AS ROW END NOT NULL,
            PERIOD FOR SYSTEM_TIME ([SysStartTime], [SysEndTime]));

        ALTER TABLE [dbo].tmp    
        SET    
        (  
        SYSTEM_VERSIONING = ON (HISTORY_TABLE = [dbo].tmpHistory)  
        ); 

        INSERT INTO dbo.tmp
        (
            Name
        )
        VALUES
        (N'Testing1'),
        (N'Testing2'),
        (N'Testing3')

        select * from tmp
     
        UPDATE tmp SET Name = 'Testing2 - 1' WHERE Name = 'Testing2'

        select * from tmp
        select * from tmpHistory

        delete from tmp WHERE Name = 'Testing2 - 1'
        
        select * from tmp
        select * from tmpHistory

    {% endhighlight %}

The first 3 selects produce:

![firstselects]/images/rowversiontemporal1.png){:class="img-responsive"}

As we can see... the row version placed into the history table on the second select is the rowversion for when the first was inserted, and is not updated when it is placed into the history table.

And the third select (i.e. the delete):
![deleteselects]/images/rowversiontemporal2.png){:class="img-responsive"}

The newly added history record contains the rowversion from the previous update statement.






