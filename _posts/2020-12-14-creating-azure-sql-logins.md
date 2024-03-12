---
layout: post
title:  "Creating Azure SQL logins & users that easily replicate to all databases."
date:   2020-12-14 12:32:00 +1000
categories: [azure, sql]
permalink: /blog/creating-azure-sql-logins-users-that-easily-replicate-to-all-databases/
---

Creating Azure SQL logins and database users is fairly simple.

First create the login on the server:

    CREATE LOGIN login_name WITH PASSWORD = 'strong_password';

Then create the user on the database and assign permissions

    CREATE USER 'user_name' FOR LOGIN 'login_name';
    EXEC sp_addrolemember 'db_datareader', 'user_name'

I became stuck on what the order of creating Azure sql server logins and then database users on the primary database and then on the secondary read only database that it replicates to.
This is because users replicate but logins do not. While at the same time I could create logins on the read only replicant but could not create users.

Thinking about it now the simple solution may be to create the logins on both servers and then create the user on the primary so that it replicates. What happens when you bring up another replication location and forget to create the login first?

A simpler solution is to create Contained users which replicate with their password.

    CREATE USER user_name WITH PASSWORD = 'strong_password';
    EXEC sp_addrolemember 'db_datareader', 'user_name'


When you refresh passwords in future this can be done on the primary only and have it replicate:

    ALTER USER user_name WITH PASSWORD = 'strong_password';

Making for a much simpler solution that will easily scale when new replicates are brought online.

