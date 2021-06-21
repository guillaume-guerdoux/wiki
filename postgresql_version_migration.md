**Migrate postgres from 10 to 12**
----
_This tutorial shows how to migrate postgres from 10 to 12_

[Link 1](https://stackoverflow.com/questions/60409585/how-to-upgrade-postgresql-database-from-10-to-12-without-losing-data-for-openpro)

### First create a backup of all the databases for that (You can continue from B if you dont need a backup)
Log in as postgres user
```
sudo su postgres
```
Create a backup .sql file for all the data you have in all the databases
```
pg_dumpall > backup.sql
```

### Upgrade to PostgreSQL12

update packages and install postgres 12
```
sudo apt-get update
sudo apt-get install postgresql-12 postgresql-server-dev-12
```
Stop the postgresql service
```
sudo systemctl stop postgresql.service
```

migrate the data
```
/usr/lib/postgresql/12/bin/pg_upgrade \
--old-datadir=/var/lib/postgresql/10/main \
--new-datadir=/var/lib/postgresql/12/main \
--old-bindir=/usr/lib/postgresql/10/bin \
--new-bindir=/usr/lib/postgresql/12/bin \
--old-options '-c config_file=/etc/postgresql/10/main/postgresql.conf' \
--new-options '-c config_file=/etc/postgresql/12/main/postgresql.conf'
```
Switch to regular user
```
exit
```
Swap the ports the old and new postgres versions.
```
sudo vim /etc/postgresql/12/main/postgresql.conf
#change port to 5432
sudo vim /etc/postgresql/10/main/postgresql.conf
#change port to 5433
```
Start the postgresql service
```
sudo systemctl start postgresql.service
```
Log in as postgres user
```
sudo su postgres
```
Check your new postgres version
```
psql -c "SELECT version();"
```
Run the generated new cluster script
```
./analyze_new_cluster.sh
```
Return as a normal(default user) user and cleanup up the old version's mess
```
sudo apt-get remove postgresql-10 postgresql-server-dev-10
#uninstalls postgres packages
sudo rm -rf /etc/postgresql/10/
#removes the old postgresql directory
sudo su postgres
#login as postgres user
./delete_old_cluster.sh
#delete the old cluster data
```
Congrads! Your postgresql version is now upgraded, If everything works well in B, we dont have to apply the backup as we have already migrated the data from the older version to the newer version, the backup is just in case if anything goes wrong.

NOTE: Change the postgresql.conf and pg_hba.conf as per your requirement
