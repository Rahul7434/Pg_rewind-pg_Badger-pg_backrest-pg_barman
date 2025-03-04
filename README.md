# Pg_rewind-pg_Badger-pg_backrest-pg_barman


# Pg_rewind
```
pg_rewind is a Postgresql utility used to bring back old primary as standby after failvour without full base backup.

- Compares the timeline history between the target server (old primary) and the source server (current primary).
- Identifies the divergence point in the WAL files & Copies only the nessary data blocks and WAL changes from the source tothe target.
- Syncs the Data Directories of the target server with the current primary.
```
---
#### When to Use pg_rewind?
```
1.After a failover or switchover in streaming replication.
2.When the primary server becomes out or sync due to crash recovery.
3.To avoid a full pg_basebackup in high availability configurations.
```
#### Setup for pg_rewind 
```
- Must have WAL archiving on both servers & Streaming Replication Enabled.
- need super user access
- The old Primary (target) server must be shut down propely before running pg_rewind.
- wal_log_hints = on --(This parameter needs to be on on the new primary server to identify the changes between two server.)
```
#### Configuration Steps
- Change postgresql.conf file on Standby (New Primary):
```
wal_level = replica
archive_mode= on
archive_command = 'cp %p /path/archivelocation/%f'
max_wal_sender = 5
wal_keep_size = 64MB
hot_standby = on
```
- Restart the Primary server
```
systemctl restart postgresql
```
- Check the Divergence Point (on the current primary new primary)
```
SELECT pg_current_wal_lsn();
SELECT timeline_id FROM pg_control_checkpoint();
```
- Shout Down the old Primary (Target)
```
Systemctl stop postgresql
```
- Run pg_rewind 
```
pg_rewind -- targe-pgdata=/var/lib/pgsql/16/data
--source-server="host=enter_ip, port= enter_port, user=postgres, password=enter_password"
```
- Create standby.signal file on (old Primary (target)).
```
cd /var/lib/pgsql/16/data
touch standby.signal

Add connection details in postgresql.conf
primary_conninfo = 'host= ip, port=port, user=username, password=password'


```

- Start the old primary as standby
```
systemctl startpostgresql
```




