# Seafile-13 Updating Guide (unRAID)

## Updating to version 13
Your seafile container may have trouble connecting to your existing database.  Check your ``seafile.conf`` file for the current SQL password.  Copy and paste the SQL password in the new template under ``SEAFILE_MYSQL_DB_PASSWORD``
