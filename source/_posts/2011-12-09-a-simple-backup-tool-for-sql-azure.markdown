---
layout: post
title: "A Simple backup tool for SQL Azure"
date: 2011-12-09
comments: true
categories: .NET
---

If you are using SQL Azure nowadays, you probably know that there are
only a few alternatives for making backups of your existing data.

-   You can use SQL Azure Data Sync, which provides bi-directional data
    synchronization and data management capabilities allowing data to be
    easily shared across SQL Azure databases within multiple data
    centers. This implies you need at least one additional database for
    backing up your data, and SQL Azure databases are not necessarily
    cheap to keep them around without any functional use.
-   You can use third party tools like the ones offered by Cerebrata or
    RedGate to synchronize SQL Azure databases with On-Premises
    databases or dump your data to files

Or you can use
[Hgdbackup](http://weblogs.asp.net/bleroy/archive/2011/12/04/source-controlled-database-backups.aspx),
an OS alternative started by [Bertrand Le
Roy](http://weblogs.asp.net/bleroy) that uses the SQL BCP command line
tool to persist your database data into text-based files.  As this was
exactly what I was needing for an Azure project, I decided to contribute
some code to that tool for storing the backup files to the Blob Storage
in Azure. Therefore, you can now specify in the tool the Azure storage
account you want to use to store your backup files, or restore your
database from. 

The tool uses the raw REST API for the Blob Storage service so the Azure
SDK or the binaries included on it are not required. This means the tool
is very simple to distribute or run in a server with minimal requisites.

The following examples illustrates how you can run the tool for making a
new backup or restore an existing database.

**Backup Example**

Hgdbackup.exe /S:tcp:YOUR\_Server.database.windows.net /D:YOUR\_DB
/U:YOUR\_USER /P:YOUR\_PASSWORD /A:YOUR\_BLOB\_ACCOUNT
/K:YOUR\_BLOB\_KEY /T:"Server={0},1433;Database={1};User
Id={2};Password={3};Trusted\_Connection=False;Encrypt=True;"

**Restore Example**

Hgdrestore.exe /S:.\\SQLExpress /D:YOUR\_DB  /C:YOUR\_BLOB\_CONTAINER
/A:YOUR\_BLOB\_ACCOUNT /K:YOUR\_BLOB\_KEY

The backup tool supports an additional argument “C:” for passing the
container where you want to store the text files with the data. If you
don’t provide a container name, the tool will generate a new container
for you.

The good thing about this tool is that it is an open source initiative
so you are free to contribute, give feedback or improve it as I did with
this support for the blob storage.

