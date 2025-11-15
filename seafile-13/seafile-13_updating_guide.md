# Seafile-13 Updating Guide (unRAID)

This file will be used to keep track of changes need to be performed manually when updating existing installations.

## Updating to version 13
Seafile-13 changed a couple things regarding the database setup.  Previously, as long as the seafile container could connect to the SQL database it would make all the required tables, users, etc.  In version 13 however, they need to be defined in our ENV variables.  I have setup the template to already have everything pre-filled in so this should not be a problem.

The only issue I ran into when upgrading to version 13 from version 12 on my test server was that Seafile-12 automatically generated the database user and their respective password.  You will need to find the current database user password in your ``seafile.conf`` file and the ncopy and pase it in the new template under ``SEAFILE_MYSQL_DB_PASSWORD``.
