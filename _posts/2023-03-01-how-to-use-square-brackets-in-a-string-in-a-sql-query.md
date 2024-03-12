---
layout: post
title:  "How to use square brackets '[]' in a string in a SQL query"
date:   2023-03-01 19:00:00 +1000
categories: [programming, databases]
permalink: /blog/how-to-use-square-brackets-in-a-string-in-a-sql-query/
---

In SQL, square brackets play a significant role in various scenarios, including escaping table and column names. However, due to their special meaning, SQL Server doesn't interpret them as usual when they appear within a string. This article aims to shed light on the usage of square brackets in SQL and provide a practical example of escaping them within a string.

### Understanding Square Brackets in SQL:

Square brackets serve multiple purposes in SQL, with their primary role being the delimiting of identifiers. Identifiers enclosed within square brackets are treated as literal values, allowing the use of reserved keywords or special characters as table or column names. For instance, consider a table named [Table] or a column named [Column]. By enclosing them in square brackets, SQL Server recognizes them as identifiers rather than reserved keywords.

### Escaping Square Brackets within a String:

When square brackets are part of a string value, they do not retain their special meaning. To include square brackets as literal characters in a string, they need to be escaped. Here's an example illustrating the process:


    SELECT * FROM TableName WHERE SomeProperty LIKE '[[]TEST]%'

In the above SQL query, the LIKE operator is used to search for rows where the property 'SomeProperty' starts with the string '[TEST]'. However, since square brackets have a special meaning in SQL, they need to be escaped by doubling them within the string. In this case, '[[]' represents the literal character '['.