---
layout: post
title: "Parameterized Queries (Oracle, SQLServer, OleDb)"
date: 2006-09-28
comments: true
categories: .NET
---

In my opinion, the parameterized queries approach is always better than
the traditional approach of joining string to build a dynamic SQL
string.

This last one is susceptible to SQL injections attacks, and usually
leads to data format problems, you have to worry about how to encode the
parameter according to its type. (For instance, the string parameters
must be enclosed with a single quote).

Unfortunately, each database vendor implements parameterized queries in
different ways, and it took me sometime today to figure out how to
use them in Oracle.

SQL server uses a named parameter approach, it does not matter the
parameters order, but each parameter name requires a "@" prefix.

**string sql = "SELECT \* FROM Customers WHERE CustomerId =
@CustomerId";**

**SqlCommand command = new SqlCommand(sql);**

**command.Parameters.Add(new SqlParameter("@CustomerId",
System.Data.SqlDbType.Int));**

**command.Parameters["@CustomerId"].Value = 1;**

Oracle uses a similar approach, but a different prefix, since it expects
a ":" prefix.

**string sql = "SELECT \* FROM Customers WHERE CustomerId =
:CustomerId";**

**OracleCommand command = new OracleCommand(sql);**

**command.Parameters.Add(new OracleParameter(":CustomerId",
OracleType.Int32));**

**command.Parameters[":CustomerId"].Value = 1;**

OleDB uses a sequential approach, in this case, the parameters
order really matters. A parameter is identified by a character "?" in
the SQL query.

**string sql = "SELECT \* FROM Customers WHERE CustomerId = ?";**

**OleDbCommand command = new OleDbCommand(sql);**

**command.Parameters.Add(new OleDbParameter("CustomerId",
OleDbType.Integer));**

**command.Parameters["CustomerId"].Value = 1;**

 

Any comment is welcome.

Pablo. 

