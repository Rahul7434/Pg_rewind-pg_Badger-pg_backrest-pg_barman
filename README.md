# Pg_rewind-pg_Badger-pg_backrest-pg_barman


# Pg_rewind
---
pg_rewind is a tool used to bring back old primary as standby after failvour without full base backup.

It is a tool compares the data on both servers, finding any changes or diverging data, it then rewind the changes on standby server.
It uses wal files to catch up the chbages thta happened on the priumary server after the failvour point.

---
- Setup for pg_rewind 
```
- Ensure you have a running replication setup
- Must have WAL archiving on
- need super user access
- on the standby server stop the postgresql
- 



