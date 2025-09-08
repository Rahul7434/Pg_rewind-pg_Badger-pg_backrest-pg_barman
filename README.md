# Pg_rewind | repmgr | pg_Badger | pg_backrest | pg_barman | pg_gather

# Pg_rewind
```
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
# pg_backrest

```
pgBackRest is a reliable backup and restore tool for PostgreSQL.
The guide is sequential: each section builds on the previous one (e.g., Restore depends on Quick Start).
It’s designed for Debian/Ubuntu + PostgreSQL 16, but adaptable to other Unix systems.
Commands are tested on virtual machines, so examples are verified and trustworthy.
You should run commands as a non-root user with sudo access to both root and postgres.
```
---
---
# 🛠️ PostgreSQL Diagnostic Toolkit: pgBadger & pg_gather
```
pgBadger is a PostgreSQL log analyzer that reads database logs and generates detailed, visual reports about performance, errors, query behavior, and more.
It’s written in Perl, a powerful scripting language known for text processing.
It parses PostgreSQL logs and produces HTML reports with graphs and summaries.
It helps us to identify slow queries, monitor activity, and troubleshoot issues,monitor connection spikes, and audit system behavior
It does not require direct access to DB, it works entirely from log files.
```
```
## 🔍 What pgBadger Can Reveal

### 🐢 Slow Queries
- Shows queries that exceed a certain duration threshold.
- Highlights execution time, frequency, and source (user, database, client IP).
- Helps you pinpoint performance bottlenecks.

### 🔁 Repeated Queries
- Aggregates identical queries and shows how many times each was run.
- Useful for identifying inefficient query patterns or missing caching.

### 🔢 Total Query Count
- Displays total number of queries executed per hour/day.
- Breaks down by database, user, and application.

### 🔒 Blocked Queries / Lock Waits
- Detects queries that were blocked due to locks.
- Shows wait duration, affected tables, and conflicting sessions.
- Requires `log_lock_waits = on` in `postgresql.conf`.

### 🧹 Autovacuum Activity
- Logs when autovacuum runs and on which tables.
- Shows duration and effectiveness.
- Requires `log_autovacuum_min_duration` to be set (e.g., `0` to log all).

### 🧱 Checkpoint Events
- Reports how often checkpoints occurred.
- Shows write and sync times, WAL file sizes.
- Requires `log_checkpoints = on`.

### 🔌 Connection Stats
- Tracks connections and disconnections.
- Shows spikes in session counts, client IPs, and failed login attempts.
- Requires `log_connections = on` and `log_disconnections = on`.

### ⚠️ Errors & Warnings
- Captures syntax errors, permission issues, deadlocks, and failed queries.
- Helps with debugging and auditing.

### 📊 Performance Graphs
HTML report includes:
- Query duration histogram
- Connections over time
- Checkpoint frequency
- Lock wait distribution

### 🧠 Bonus Insights (If Logging Is Enabled)
- Temporary file usage (indicates memory pressure)
- Statement types breakdown (SELECT, INSERT, UPDATE, etc.)
- Top 10 slowest queries
- Top 10 most frequent queries
- Application-level stats (if `application_name` is set)
```

---

## 📦 1. pgBadger — PostgreSQL Log Analyzer

### 🔧 Installation Steps

#### ✅ Prerequisites
- PostgreSQL must be configured to log queries.
- Perl must be installed (pgBadger is written in Perl).

#### 🖥️ Install pgBadger (Linux)
```bash
sudo apt install pgbadger   # Debian/Ubuntu
sudo yum install pgbadger   # RHEL/CentOS
```

#### 🧰 Configure PostgreSQL Logging
Edit `postgresql.conf`:
```conf
logging_collector = on
log_directory = 'pg_log'
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_statement = 'mod'       # or 'all' for full query logging
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '
log_duration = on
```

Restart PostgreSQL:
```bash
sudo systemctl restart postgresql
```

---

### 📊 Generate Report
```bash
pgbadger /var/lib/pgsql/pg_log/*.log -o /tmp/pgbadger_report.html
```

#### 🔍 Optional Flags
```bash
-j 4                 # Use 4 CPU cores for parallel parsing
-f stderr            # Specify log format
-b "2025-09-01"      # Start date
-e "2025-09-07"      # End date
```

---

### 🔄 Backend Workflow

1. PostgreSQL writes logs to `pg_log/` based on your config.
2. pgBadger parses these logs line by line.
3. It extracts:
   - Query durations
   - Errors and warnings
   - Checkpoint and vacuum activity
   - Connection stats
4. It builds an HTML report with graphs and summaries.

📌 **No direct DB access** — it works entirely from log files.

---

## 🧠 2. pg_gather — SQL-Based Health Snapshot Tool

### 🔧 Installation Steps

#### ✅ Prerequisites
- No installation needed — just download the SQL scripts.
- Requires access to `psql` and a PostgreSQL user with read privileges.

#### 📥 Download Scripts
From the official repo or trusted source:
- `gather.sql`
- `gather_report.sql`
- `history_schema.sql` (optional for storing snapshots)

---

### 📊 Run Data Collection
```bash
psql -U postgres -d mydb -f gather.sql > /tmp/db_snapshot.tsv
```

### 📊 Generate Report
Import snapshot into analysis DB:
```bash
psql -U postgres -d analysis_db -f history_schema.sql
\copy gather_data FROM '/tmp/db_snapshot.tsv' WITH CSV
psql -U postgres -d analysis_db -f gather_report.sql
```

---

### 🔄 Backend Workflow

1. `gather.sql` runs SQL queries against system catalogs:
   - `pg_stat_user_tables`, `pg_stat_statements`, `pg_class`, `pg_index`
2. It collects:
   - Table bloat
   - Index usage
   - Autovacuum stats
   - Configuration parameters
   - Replication lag
3. Output is saved as TSV for portability.
4. `gather_report.sql` analyzes the snapshot and generates insights.

📌 **No external dependencies** — works entirely via SQL.
---
## 🧪 Combined Workflow Example

| Task                        | Tool        | Command / Action                                      |
|-----------------------------|-------------|--------------------------------------------------------|
| Weekly health audit         | pg_gather   | `psql -f gather.sql > snapshot.tsv`                   |
| Analyze unused indexes      | pg_gather   | `gather_report.sql`                                   |
| Daily query performance     | pgBadger    | `pgbadger *.log -o report.html`                       |
| Troubleshoot slow queries   | pgBadger    | Filter by duration, error, or connection spikes       |
| Monitor autovacuum behavior | pg_gather   | Check `pg_stat_user_tables` and vacuum thresholds     |
---


