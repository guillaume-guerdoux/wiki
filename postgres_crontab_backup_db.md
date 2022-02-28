**Use automatic local backup for postgres**
----
_This tutorial shows how to add crontab to create automatic backup for postgres

```
0 0 * * * pg_dump -U postgres DB_NAME_db > ~/backups/DB_NAME_db.bak
```
