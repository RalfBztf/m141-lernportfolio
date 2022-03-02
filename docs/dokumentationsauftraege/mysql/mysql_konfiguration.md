# MySQL Konfiguration


## Storage Engines

```InnoDB und MyISAM``` sind die 2 wichtigsten Storage Engines von MySQL. Ein Storage Engine ist dazu da, wie die Daten auf der Festplatte oder im RAM gepseichert werden. 

### InnoDB

Bei MySQL ist die verbreiteste und eine der wichtigsten die InnoDB Engine. Sie unterstützt [Transaktionen](dokumentationsauftraege/mysql/begriffe_historisches?id=transaktionen), ist komplett [ACID Konform](dokumentationsauftraege/mysql/begriffe_historisches?id=acid) und unterstützt die Sperrung auf Zeilenebende, damit Ein Datensatz nicht gleichzeitig gelesen/bearbeitet wird. Es ist die einzige Engine die Integrität für relationen (Fremdschlüssel) bietet.

### MyISAM

MyISAM baut auf ISAM (nicht mehr supported) auf. Es war die frühere Engine von MySQL. Allerdings unterstützt sie kein oder nicht komplett das ACID Konzept. Sie bietet jedoch sperre auf Tabellenebene an. Es können keine Transaktionen gemacht werden und es bestehen keine Beziehungen (Fremdschlüssel) in andere Tabellen. Die Engine ist sehr simple, geeignet für Fulltext Search in der DB (InnoDB kann dies heute jedoch auch) und ist performant wenn man hauptsächlich nur Lesen muss und die Datenintegrität nicht so wichtig ist.


### Andere Engines

MySQL-Shell starten:

```bash
sudo mysql
```

Engines anzeigen:

```sql
show engines;
```

#### CSV

Mit der CSV-Engine werden die Daten im CSV-Format (Komma separiert) gespeichert. Dies ist nützlich wenn die Daten mit anderen Applikation (z.B Excel) geteilt/eingelesen werden müssen.

#### ARCHIVE

Mit der ARCHIVE-Engine werden Daten nach dem speichern sofort komprimiert. Diese Engine eignet sich am besten, wenn Daten archiviert weren müssen und daher selten gelesen werden.

#### BLACKHOLE

Mit der BLACKHOLE-Engine kann man Daten sofort wieder vernichten. Verlgleichbar mit UNIX-Systemen bei denen man einen Befehl ausführt, aber den Output direkt nach /dev/null befördert. Dies braucht man vorallem beim Testing.

#### FEDERATED

Man kann auf Daten von einem Remote MySQL-Server zugreifen ohne Clustering/Replikation zu nutzen. Diese Daten werden jedoch nicht in den lokalen Tabellen gespeichert. Diese Engine ist aber per Default deaktiviert und müsste in ```/etc/mysql/my.cnf``` hinzugefügt werden.

#### Memory

Die Daten werden im RAM gespeichert und sind somit flüchtig. Die Engine ist aber sehr geeignet, wenn man temporäre Tabellen macht. Z.B beim auswerten mehrer Tabellen.

#### MRG_MYISAM

Mit der MRG_MYISAM oder auch MERGE-Engine genannt, kann man identische MYISAM Tabellen zusammenführen. Identisch heisst, dass die Spaltendatentypen und Indexinformationen (z.B Spaltenreihenfolge) genau gleich sein muss.

#### PERFORMANCE_SCHEMA

In MySQL gibt es eine Datenbank ```performance_schema``` welche die Storage-Engine PERFORMANCE_SCHEMA nutzt. Sie ist eine In-Memory Datenbank (wird gelöscht nach Server neustart) und ist zuständig für minimale Überwachung des Datenbank servers.  Man kann in der Tabelle die ganzen Informationen mit einfachen SELECTs abfragen. Beispiele dafür sind:

* Errors
* Variablen
* Infos zu Events (z.B SQL-Asuführungen)
* und vieles mehr

Diese Engine ist in den neueren Version per Default aktiviert und müsste in ```/etc/mysql/my.cnf``` deaktiviert werden, falls man diese nicht verwenden will.

### Anwendung einer Engine

Die Anwendung einer Engine auf eine Tabele funktioniert wie folgt: 

Inauen fragen, verstehe frage nicht ganz.


## Benutzer und Berechtigungen

### Demo DB

MySQL-Shell starten:

```bash
sudo mysql
```

Demodatenbank mit 2 Tabellen erstellen:

```sql
CREATE DATABASE demoDB;
USE demoDB;
CREATE TABLE Personen(Vorname varchar(50), Nachname varchar(50));
CREATE TABLE Produkte(name varchar(100), preis int);
```

Tabellen und Attributen anzeigen:

```sql
SHOW TABLES;
DESCRIBE Personen;
DESCRIBE Produkte;
```

Der ```root``` Benutzer wurde bereits bei der Installation richtig konfiguriert (Passwort gesetzt und Zugriff nur von localhost). Für die Nutzung der Datenbank mit einer Appliaktion sollte aber ein eigener Benutzer verwendet werden.

Benutzer erstellen:

```sql
CREATE USER 'webapp01'@'localhost' IDENTIFIED BY 'Skjh9w874!';
```

Berechtigungen anpassen:

```sql
GRANT CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on *.* TO 'webapp01'@'localhost';
```

Admin-User erstellen:

```sql
CREATE USER 'admin01'@'localhost' IDENTIFIED BY 'Skjh9w874!';
```

Berechtigungen setzen (Admin darf alles):

```sql
GRANT ALL PRIVILEGES ON *.* TO 'admin01'@'localhost' WITH GRANT OPTION;
```

Berechtigungen übernehmen (falls noch irgendwo etwas gecached ist):

```sql
FLUSH PRIVILEGES;
```

Mit neuem User einloggen:

```bash
mysql -u webapp01 -p
```

Danach wird man aufgefordert das Passwort einzugeben und die MySQL-Shell öffnet sich. Wir sind nun berechtigt z.B Datenbanken zu erstellen, bearbeiten, löschen, aber dürfen anderen User keine Berechtigungen erteilen.

![webapp01_shell](webapp01_shell.png)

Mit Adminuser einloggen:

```bash
mysql -u admin01 -p
```

Danach auch wieder das Passwort eingeben. Nun versuchen wir noch ein User zu erstellen, welcher nur die 4 CRUD Operation ausführen darf. Dies sollte funktionieren, da der ```admin01``` auch Berechtigungen erteilen darf.

```sql
CREATE USER 'webapp02'@'localhost' IDENTIFIED BY 'Skjh9w874!';
```

Berechtigungen anpassen:

```sql
GRANT INSERT, UPDATE, DELETE, SELECT on *.* TO 'webapp02'@'localhost';
```

![webapp01_shell](admin01_shell.png)


---------

    Server-Konfiguration
        Transaktions-Isolation : Verfizieren Sie welche Transaktions-Isolation auf Ihrem Server aktiviert ist. Dokumentieren Sie, was das bezüglich den Anomalien für Ihre Installation bedeutet.
        Exportieren Sie die aktuelle Liste an System-Variablen
        Netzwerkkonfiguration DBMS-Server anpassen und dokumentieren

    Server-Betrieb
        Protokollierung langsamer Abfragen aktivieren
        Listen Sie den Inhalt des Data-Directories auf
        Dokumentieren Sie die Default-Datenbanken mysql, sys, performance_schema



    (Tag4) Storage Engines bei MySQL
    (Tag4) Benutzer und Berechtigungen
    (Tag4) Server-Konfiguration
    (Tag4) Server-Betrieb
