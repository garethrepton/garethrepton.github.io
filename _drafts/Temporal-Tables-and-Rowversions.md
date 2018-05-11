---
layout: post
title: SQL Server Temporal Tables and Rowversions
tags: sqlserver rowversions temporal tables
---

This is more of a note than a full post. I was intruiged to see how a table with System_Versioning enabled would behave if it had a rowversion on the primary table, and how it would get inserted to the history table. 

The short answer is that it copies the rowversion of the previous change to the record into the history table. This feels slightly counterintuitive from a rowversion perspective as every change to a row will generate a new database level rowversion and set it on that record. But it is inkeeping with how temporal tables work (i.e. theres no state except the start/end dates so a record is copied exactly into the history table).

It should also be noted that there doesnt appear to be a way to add a rowversion to the history table either. I.e. it must match the primary tables columns, so no you can't just st it the other column to varbinary and add a new rowversion column.

## Heres the Proof