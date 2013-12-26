---
layout: post
title: "Multitenancy in SQL Azure"
date: 2012-12-06
comments: true
categories: .NET
---

If you are building a SaaS application in Windows Azure that relies on
SQL Azure, it’s probably that you will need to support multiple tenants
at database level.

This is short overview of the different approaches you can use for
support that scenario,

**A different database per tenant**

A new database is created and assigned when a tenant is provisioned.

Pros

-   Complete isolation between tenants. All the data for a tenant lives
    in a database only he can access.

Cons

-   It’s not cost effective. SQL Azure databases are not cheap, and the
    minimum size for a database is 1GB.  You might be paying for storage
    that you don’t really use.
-   A different connection pool is required per database.
-   Updates must be replicated across all the databases
-   You need multiple backup strategies across all the databases

**Multiple schemas in a database shared by all the tenants**

A single database is shared among all the tenants, but every tenant is
assigned to a different schema and database user.

Pros

-   You only pay for a single database.
-   Data is isolated at database level. If the credentials for one
    tenant is compromised, the rest of the data for the other tenants is
    not.

Cons

-   You need to replicate all the database objects in every schema, so
    the number of objects can increase indefinitely.
-   Updates must be replicated across all the schemas.
-   The connection pool for the database must maintain a different
    connection per tenant (or set of credentials)
-   A different user is required per tenant, which is stored at server
    level. You have to backup that user independently.

**Centralizing the database access with store procedures in a database
shared by all the tenants**

A single database is shared among all the tenants, but nobody can read
the data directly from the tables. All the data operations are performed
through store procedures that centralize the access to the tenant data.
The store procedures contain some logic to map the database user to an
specific tenant.

Pros

-   You only pay for a single database.
-   You only have a set of objects to maintain and backup.

Cons

-   There is no real isolation. All the data for the different tenants
    is shared in the same tables.
-   You can not use traditional ORM like EF code first for consuming the
    data.
-   A different user is required per tenant, which is stored at server
    level. You have to backup that user independently.

**SQL Federations**

A single database is shared among all the tenants, but a different
federation is used per tenant. A federation in few words, it’s a
mechanism for horizontal scaling in SQL Azure, which basically uses the
idea of logical partitions to distribute data based on certain criteria.

Pros

-   You only have a single database with multiple federations.
-   You can use filtering in the connections to pick the right
    federation, so any ORM could be used to consume the data.

Cons

-   There is no real isolation at that database level. The isolation is
    enforced programmatically with federations.


