# MySQL Replikation

## Vorbereitung

Repository klonen:

```bash
git clone https://github.com/garutilorenzo/mysql-innodb-cluster
```

### Netzwerkplan

### Configfiles

#### Master

```conf
[mysqld]

server-id = 1
port = 3306

binlog_format = ROW

gtid_mode=ON
enforce-gtid-consistency=true

log-slave-updates
log_bin = mysql-bin

default_storage_engine = InnoDB

# replication

report-host = mysql_node01
slave_net_timeout = 60

skip-slave-start

transaction_isolation = 'READ-COMMITTED'

binlog_checksum = NONE
relay_log_info_repository = TABLE
transaction_write_set_extraction = XXHASH64

auto_increment_increment = 1
auto_increment_offset = 2

binlog_transaction_dependency_tracking = WRITESET 
slave_parallel_type = LOGICAL_CLOCK
slave_preserve_commit_order = ON
```

Den Servern muss man eine ID geben, um sie zu identifizieren. Port sollte per Default auf 3306 laufen. Das Binary-Log Format ROW heisst, dass es notiert, was geändert wurde, generiert aber eher grössere Logs. Für die Datenkonsistenz ist dies aber sehr wichtig, denn wenn z.B ```Statement-based``` verwendet wird und die ersten 1000 Datensätze auf dem Master gelöscht werden, aber die ersten 1000 Datensätze auf den Replicas andere sind, löscht man falsche Daten. Die Storage ist InnoDB, wobei für solche Replicas auch gut Network Database (NDB) verwendet werden könnte. ```report-host``` ist der Hostname des Replicas. Als Isolationslevel wird hier ```READ-COMMITTED``` verwendet. Das heisst es sind ```Lost Update, Non-Repeatable Read und Phantom Read``` möglich. Mit ```binlog_transaction_dependency_tracking```, ```slave_parallel_type``` und ```slave_preserve_commit_order``` wird ein Mechanismus erzeugt, damit bei jeder Transaktion ein Zeitstempel und Hashwert angefügt wird, damit alles nacheinander ausgeführt und überprüft werden kann, dass die Daten korrekt sind. Als Hashalgorithmus wird ```XXHASH64``` verwendet.

#### Slave

```conf
[mysqld]

server-id = 2
port = 3306

binlog_format = ROW

gtid_mode=ON
enforce-gtid-consistency=true

log-slave-updates
log_bin = mysql-bin

default_storage_engine = InnoDB

# replication

report-host = mysql_node02
slave_net_timeout = 60

skip-slave-start
read_only

transaction_isolation = 'READ-COMMITTED'

binlog_checksum = NONE
relay_log_info_repository = TABLE
transaction_write_set_extraction = XXHASH64

auto_increment_increment = 1
auto_increment_offset = 2

binlog_transaction_dependency_tracking = WRITESET
slave_parallel_type = LOGICAL_CLOCK
slave_preserve_commit_order = ON
```

Ziemlich gleich, ausser das sich ```server_id``` und```report-host``` geändert hat, sowie der Slave ```read-only``` ist.

## Installation

Dockercontainer mittels Docker-Compose im Hintergrund starten:

```bash
docker-compose up -d
```

Testen, ob alle Container sauber funktionieren:

```bash
docker-compose ps
```

```output

```

```bash
docker-compose logs -f
```

```output

```
