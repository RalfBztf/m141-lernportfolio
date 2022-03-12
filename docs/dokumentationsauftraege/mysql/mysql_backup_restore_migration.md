# MySQL Backup und Restore

## Import / Export Arten

* **Cold:** Dem Benutzer stehen die Daten gar nicht zur Verfügung. Dieses Backup liefert dafür praktisch 100% konsistente Daten, da diese während des Backups nicht mehr verändert werden können. (Dienst muss gestoppt sein)
* **Warm:** Dem Endbenutzer stehen die Daten teilweise zur Verfügung, um das Backup relativ sicher durchführen zu können. Der User kann z.B nur lesen.
* **Hot:** Der User kann mit den Daten während des Backups alles machen (Z.B lesen, hinzufügen, ändern). Hier besteht die Gefahr, dass das Backup nicht konsistent ist, da vielleicht etwas verändert wurde, aber im Backup nur teilweise vorhanden ist.

* **logisch:** Das Backup ist in menschenlesbarer Form z.B einer CSV- oder SQL-Datei.
* **physisch:** Das Backup ist in Binärer Form und funktioniert komplett oder gar nicht. Dies kann auch ein Snapshot einer VM sein. Man kann nicht wie beim logischen Backup einzelne Daten verändern.

* **Full:** Alle Daten werden gesichert
* **Incremental** Nach einem Fullbackup werden nur noch die Änderungen von Datenbeständen gesichert. So wird weniger Speicher beim Backup benötigt.

## Backup

### PITR

Mithilfe vom inkrementellen Backup ist ein ```Point-in-Time-Restore (PITR)``` möglich. Dazu müssen aber noch die Binären Logdaten aktiviert werden. Diese speichern alle Transaktionen.

MySQL Konfiguration bearbeiten: ```/etc/mysql/mysql.conf.d/mysqld.cnf```

```bash
[mysqld]
 server_id=1
 log-bin=mysql-bin

# Optionen
expire_logs_days = 10
max_binlog_size = 100
```

ServerID auslesen:

```mysql
select @@server_id;
```

### MySQL Backup Skript mithilfe von Bash

```bash
#!/bin/bash
DATE=$(date +%d-%m-%Y)
BACKUP_DIR="/tmp/test-backup"
MYSQL_USER='admin01'
MYSQL_PASSWORD='Skjh9w874!'
MYSQL=/usr/bin/mysql
MYSQLDUMP=/usr/bin/mysqldump

# To create a new directory in the backup directory location based on the date
mkdir -p $BACKUP_DIR/$DATE

# To get a list of databases
databases=`$MYSQL -u$MYSQL_USER -p$MYSQL_PASSWORD -e "SHOW DATABASES;" | grep -Ev "(Database|information_schema)"`

# To dump each database in a separate file
for db in $databases; do
echo $db
$MYSQLDUMP --single-transaction --force --opt --skip-lock-tables --user=$MYSQL_USER -p$MYSQL_PASSWORD --databases $db | gzip > "$BACKUP_DIR/$DATE/$db.sql.gz"
done

# Delete the files older than 10 days
find $BACKUP_DIR/* -mtime +10 -exec rm {} \;
```

```grep -Ev``` ist eine Regular Expression welche ```Database``` und ```information_schema``` exkludiert aus dem ```SHOW DATABASES;```-Befehl.

```--single-transaction``` Verhindert inkonsistente (Hot-)Backups.

```--opt``` (Options) ist ein Abkürzung für die ganzen folgenden Parametern: ```--add-drop-table --add-locks --create-options --disable-keys --extended-insert --lock-tables --quick --set-charset```

### mydumper

Nutzen von 6 Threads während dem Backup und Backup komprimieren:

```bash
mydumper -u admin01 -p Skjh9w874! --threads 6 --database pokemon --compress --outputdir /tmp/mydumper/
```

### Percona XtraBackup

Kann ein Hot Backup für produktiv Umgebungen machen. Dass heisst, dass Benutzer uneingeschränkt weiterarbeiten können. Dies aber nur für die ```InnoDB``` and ```XtraDB``` Storage Engine. Die MyISAM kann anscheinend nur im Read-Only Modus gesichert werden. Dieses Tool ist OpenSource, Enterprise ready und sehr gut automatisierbar (auch Restore Tests).

## Restore

### Restore von Pokemon (Bash Skript)

```bash
gunzip /tmp/test-backup/11-03-2022/pokemon.gz && mysql -u admin01 -pSkjh9w874! < /tmp/test-backup/11-03-2022/pokemon.sql
```

### Restore von Pokemon (myloader)

```bash
myloader -u admin01 -p Skjh9w874! --threads 6 --directory /tmp/mydumper/
```

Habe zum Test die Datenbank Pokemon bei beiden Methoden gedroppt und wieder importiert, dies ist bei mir erfolgreich verlaufen.

### Point-in-Time-Restore

Konvertierung von binärer Datei in menschenlesbares SQL-File:

```bash
mysqlbinlog mysql-bin.010310 > mysql-bin.010310.sql
```

Alle Transaktionen vor dem Stopdatum werden nicht berücksichtigt:

```bash
mysqlbinlog \
--stop-datetime = ’2019-04-29 17:00:00’ \
mysql-bin.010312 > mysql-bin.010312.sql
```

Import ganz normal:

```bash
mysql -u admin01 -pSkjh9w874! < mysql-bin.010312.sql
```

## Migration auf entfernten Server

**Hinweis:** Nach VM-Klon noch den Hostnamen ändern!

### Vorbereitung (immer ausführen, wenn eine neue Migration begonnen wird)

Auf Klonsystem (neuer Server) noch die Daten droppen und die Datenbank leer erstellen, damit man auch sieht ob die Migration erfolgreich war. (Passwort für die SSH-Verbindung wird interaktiv abgefragt)

```bash
ssh vagrant@10.0.28.5 'mysql -u admin01 -pSkjh9w874! -e "DROP DATABASE pokemon; CREATE DATABASE pokemon; USE pokemon; SHOW TABLES;"'
```

Wenn hier nur irgendwie ```mysql: [Warning] Using a password on the command line interface can be insecure.``` im Terminal ausgegeben wird heisst dies, dass keine Tabellen in der pokemon DB verfügbar sind.

### Migration mit direktem Zugriff auf aktuellen Server

Auf dem aktuellen DB-Server die Daten auf den neuen schieben (Hot-Migration):

```bash
mysqldump --single-transaction -u admin01 -pSkjh9w874! pokemon | ssh vagrant@10.0.28.5 'mysql -u admin01 -pSkjh9w874! pokemon'
```

Prüfen, ob die Daten angekommen sind mit Testquery:

```bash
ssh vagrant@10.0.28.5 'mysql -u admin01 -pSkjh9w874! -e "USE pokemon; SHOW TABLES; SELECT * FROM pokemon;"'
```

Zuerst werden die Tabellen angezeigt und danach noch die ganzen Pokemons. Migration ist also erfolgreich verlaufen.

### Migration von Laptop

Falls keine direkte Verbindung per SSH auf die Server möglich ist, müsste der erste Befehl ```mysqldump``` auch per SSH ausgeführt werden:

```bash
ssh vagrant@10.0.28.4 'mysqldump --single-transaction -u admin01 -pSkjh9w874! pokemon' | ssh vagrant@10.0.28.5 'mysql -u admin01 -pSkjh9w874! pokemon'
```

Hier kann es sein, dass die SSH-Passwortaufforderung für den 2. Server nicht kommt. Hier nach einer Zeit einfach mal Enter drücken, danach sollte sie erscheinen und die Migration kann weiter fortgesetzt werden.

Prüfen, ob die Daten angekommen sind mit Testquery:

```bash
ssh vagrant@10.0.28.5 'mysql -u admin01 -pSkjh9w874! -e "USE pokemon; SHOW TABLES; SELECT * FROM pokemon;"'
```

### Migration inkl. ZIP

Für mehr effizienz kann man das ganze natürlich noch zippen:

```bash
ssh vagrant@10.0.28.4 'mysqldump --single-transaction -u admin01 -pSkjh9w874! pokemon | gzip -9' | ssh vagrant@10.0.28.5 'gunzip | mysql -u admin01 -pSkjh9w874! pokemon'
```

Auch hier kann es sein, dass die SSH-Passwortaufforderung für den 2. Server nicht kommt. Hier nach einer Zeit einfach mal Enter drücken, danach sollte sie erscheinen und die Migration kann weiter fortgesetzt werden.

Prüfen, ob die Daten angekommen sind mit Testquery:

```bash
ssh vagrant@10.0.28.5 'mysql -u admin01 -pSkjh9w874! -e "USE pokemon; SHOW TABLES; SELECT * FROM pokemon;"'
```

### Migration mit Fifo

Mithilfe von Fifo kann man Pipes auslagern. Dies ist sehr nützlich bei instabilen Netzwerkverbindungen zum Server.

Fifo erstellen:

```bash
mkfifo migrationPipe
```

Den ganzen export in das File umleiten:

```bash
ssh vagrant@10.0.28.4 'mysqldump --single-transaction -u admin01 -pSkjh9w874! pokemon | gzip -9' > namedPipe;
```

Daten aus Fifo lesen und auf den neuen Server laden:

```bash
ssh vagrant@10.0.28.5 'gunzip | mysql -u admin01 -pSkjh9w874! pokemon' < namedPipe;
```
