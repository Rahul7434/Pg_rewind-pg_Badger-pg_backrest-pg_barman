# Pg_rewind | repmgr | pg_Badger | pg_backrest | pg_barman

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
- Run pg_rewind  on the old primary (target).
```
pg_rewind -- target-pgdata=/var/lib/pgsql/16/data
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
systemctl start postgresql
```

# repmgr
```
repmgr (Replication Manager) is a powerful open-source tool suite designed to manage replication and failover in PostgreSQL clusters.
It’s especially useful in high-availability (HA) environments where you want to ensure your database remains accessible even if one server goes down.

🧠 What is repmgr?
Developed by EDB (EnterpriseDB), repmgr enhances PostgreSQL’s built-in streaming replication.

It simplifies the setup, monitoring, and administration of primary-standby architectures.

Supports PostgreSQL versions from 9.3 to 17.

🛠️ What Can You Do with repmgr?

Feature                 Description                                                  Use Case
------------------------------------------------------------------------------------------------------------
Primary/Standby Setup   Easily configure a primary node and clone standby nodes      Initial cluster setup
Monitoring              Continuously checks replication status and node health       Detect lag, failures, or sync issues
Failover                Automatically or manually promote a standby to primary       When the primary node fails
Switchover              Gracefully switch roles between primary and standby          Planned maintenance or upgrades
Cascading Replication   Supports replication chains (standby of a standby)           Load distribution and scalability
Timeline Management     Handles timeline switches during failover                    Ensures continuity after promotion
Base Backups            Uses PostgreSQL’s replication protocol for backups           Fast and consistent standby creation
Cluster Registration    Registers nodes for coordinated management                   Keeps metadata organized
Event Notifications     Sends alerts for cluster events                              Integration with monitoring tools

🧠 What is repmgr?
Developed by EDB (EnterpriseDB), repmgr enhances PostgreSQL’s built-in streaming replication.

It simplifies the setup, monitoring, and administration of primary-standby architectures.

Supports PostgreSQL versions from 9.3 to 17.

🛠️ What Can You Do with repmgr?
Here’s a breakdown of its core capabilities:

Feature	Description	Use Case
Primary/Standby Setup	Easily configure a primary node and clone standby nodes	Initial cluster setup
Monitoring	Continuously checks replication status and node health	Detect lag, failures, or sync issues
Failover	Automatically or manually promote a standby to primary	When the primary node fails
Switchover	Gracefully switch roles between primary and standby	Planned maintenance or upgrades
Cascading Replication	Supports replication chains (standby of a standby)	Load distribution and scalability
Timeline Management	Handles timeline switches during failover	Ensures continuity after promotion
Base Backups	Uses PostgreSQL’s replication protocol for backups	Fast and consistent standby creation
Cluster Registration	Registers nodes for coordinated management	Keeps metadata organized
Event Notifications	Sends alerts for cluster events	Integration with monitoring tools

📌 When Should You Use repmgr?
Use repmgr in scenarios like:
✅ High Availability Needs: You want automatic failover to minimize downtime.

🔄 Disaster Recovery Planning: You need standby nodes ready to take over.

📊 Performance Scaling: You want to offload read queries to replicas.

🧪 Testing Failover: You want to simulate and validate HA setups.

🛠️ Routine Maintenance: You need to switch roles without downtime.

```
```
🛠️ Prerequisites (Before Configuration)
Make sure the following are in place on both master and standby servers:
PostgreSQL installed (same version across nodes)
repmgr installed (matching PostgreSQL version)
Network access between nodes (default PostgreSQL port: 5432)
SSH access between nodes (for repmgr to execute remote commands)
Time synchronization (e.g., via NTP)

⚙️ PostgreSQL Configuration
Update postgresql.conf on all nodes:
wal_level = 'replica'
max_wal_senders = 10
max_replication_slots = 10
hot_standby = on
archive_mode = on
archive_command = '/bin/true'
shared_preload_libraries = 'repmgr'

Update pg_hba.conf to allow replication and repmgr connections:
# Replication access
host    replication     repmgr     192.168.1.0/24     trust

# repmgr access
host    repmgr          repmgr     192.168.1.0/24     trust

🧑‍💻 User and Database Setup
Create a dedicated user and database for repmgr:

CREATE USER repmgr SUPERUSER;
CREATE DATABASE repmgr OWNER repmgr;

📁 repmgr.conf Configuration
Each node needs a repmgr.conf file. Key parameters include:

Parameter               Description
--------------------    ---------------------------------------------------------------
node_id                 Unique ID for each node
node_name               Descriptive name of the node
conninfo                Connection string to PostgreSQL (host=... user=repmgr dbname=repmgr)
data_directory          Path to PostgreSQL data directory
pg_bindir               Path to PostgreSQL binaries (e.g., /usr/pgsql-13/bin)
log_file                Path to repmgr log file
failover                automatic or manual
promote_command         Command to promote standby to primary
follow_command          Command to follow new primary after failover
reconnect_attempts      Number of attempts to reconnect during failover
reconnect_interval      Interval between reconnect attempts
monitoring_history      Number of monitoring records to keep
priority                Priority for failover (lower number = higher priority)

Example:
node_id=1
node_name=node1
conninfo='host=192.168.1.10 user=repmgr dbname=repmgr'
data_directory='/var/lib/pgsql/13/data'
pg_bindir='/usr/pgsql-13/bin'
log_file='/var/log/repmgr/repmgr.log'
failover=automatic
promote_command='/usr/pgsql-13/bin/repmgr standby promote -f /etc/repmgr.conf'
follow_command='/usr/pgsql-13/bin/repmgr standby follow -f /etc/repmgr.conf'
priority=100


🚀 Setup Steps
On Primary Node:
1.Register the primary:
repmgr primary register

On Standby Node:
2.Clone the standby:
repmgr standby clone --force --verbose

3.Register the standby:
repmgr standby register

🔄 Post-Setup: Enable Daemon
Start the repmgr daemon (repmgrd) on all nodes:

repmgrd -f /etc/repmgr.conf --daemonize

```



