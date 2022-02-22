# MySQL Installation

## Infos



## Recherche MySQL

Der Hersteller von MySQL ist Oracle. MySQL hat ein Dual-Lizenz-Model. Wer FOSS (Free and Open Source Software) unter der GPL Lizenz veröffentlicht, darf die OpenSource Variante von MySQL gratis nutzen. (Lizenzhinweis ist trotzdem Pflicht!). Siehe https://www.mysql.com/about/legal/licensing/oem/
Wer kommerzielle (Closed Source) Software verkaufen will und eine MySQL DB nutzt muss einen Vertrag mit Oracle eingehen und zahl dazu je nach Edition mehrere 10'000$. Siehe https://shop.oracle.com/apex/product?p1=MySQL
Dafür hat man dort dann aber aktuellen Support. (Administration Tools, versprechen für grosse Ausfallsicherheit,längere Security Patches etc.)
MySQL ist ein RDBMS (Relationales Datenbank Management System), welches sich sehr gut eignet, wenn man eben Relationen hat. Z.B User und Bestellungen oder Bestellungen und Produkte. MySQL verfolgt das Client-Server Model. Die Daten liegen auf dem mysql-server welches mit einem mysql-client abgerufen/erstellt werden können. Ca. alle 3 Monate kommt vom Server eine neue Version.

## Voraussetzungen

Ich setze ein 64-bit Ubuntu 21.04 (hirsute) mit dem Linux Kernel 5.11.0-49-generic ein.
MySQL installiere ich mittels dem Paketmanager ```apt``` in der aktuellsten Version (freigegeben für hirsute) ein, obwohl ich lieber Docker hätte. ;) Im Moment ist die für Client sowie Server die MySQL Version 8.0.27.

        Welche Dienste laufen sonst noch auf dem Server?

## Installation

Erstmals das ganze System updaten:
```bash
sudo apt update && sudo apt upgrade
```

MySQL-Server installieren, dies hat folgende Abhängigkeiten:

Irgendwelche Librarys:

* libcgi-fast-perl, libcgi-pm-perl, libevent-core-2.1-7, libevent-pthreads-2.1-7, libfcgi-bin, libfcgi-perl, libfcgi0ldbl, libhtml-template-perl, libmecab2

Irgendwas für die Japanische Sprache:

* mecab-ipadic
* mecab-ipadic-utf8
* mecab-utils,

MySQL Client, um Verbindungen zum Server herstellen zu können:

* mysql-client-8.0
* mysql-client-core-8.0 

MySQL Server selbst:

* mysql-server
* mysql-server-8.0
* mysql-server-core-8.0

```bash
sudo apt install mysql-server
```

Der MySQL-Server ist im Moment aber noch gar nicht geschützt. (z.B kein root Passwort) Deshalb kann über folgenden Befehle ein kleiner Installationsguide gestartet werden.

```bash
sudo mysql_secure_installation
```

Folgende Setting setzen:

* VALIDATE PASSWORD COMPONENT: no
* root password: ein sicheres Passwort
* Anonyme Benutzer entfernen: y
* remote root login deaktivieren: y
* test Datenbank und Zugriffe löschen: y
* Privilegien Tabelle neu laden: y
Die Privilegien Tabelle ist die Tabelle, welche sagt welcher Benutzer auf was Zugriff hat. Im Hintergrund wird der SQL Befehl ```FLUSH PRIVILEGES``` ausgeführt, damit die gelöschten User auch sicher keinen mehr Zugriff haben.

Nun ist folgende MySQL Version installiert: 
* mysql Ver 8.0.27-0ubuntu0.21.04.1 for Linux on x86_64 (Ubuntu)

Autostarten des MySQL Service:
```bash
sudo systemctl enable mysql
```

Starten des MySQL Service (falls noch nicht passiert):
```bash
sudo systemctl start mysql
```

## Testen

Überprüfen des MySQL Service:
Status des MySQL Service:
```bash
sudo systemctl status mysql
```
Sollte Grün sein!!!!!!!!!!!!!!!!! printscreen

```bash
sudo mysqladmin -o -u root version
```

Sollte folgendes ausgeben:
```output 
mysqladmin  Ver 8.0.27-0ubuntu0.21.04.1 for Linux on x86_64 ((Ubuntu))
Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Server version		8.0.27-0ubuntu0.21.04.1
Protocol version	10
Connection		Localhost via UNIX socket
UNIX socket		/var/run/mysqld/mysqld.sock
Uptime:			32 min 29 sec

Threads: 2  Questions: 16  Slow queries: 0  Opens: 131  Flush tables: 3  Open tables: 50  Queries per second avg: 0.008
```

MySQl-Shell starten:
```bash
sudo mysql
```

Datenbanken anzeigen:
```sql
show databases;
```

Sollte die Datenbanken ausgeben. Printscreen!!!!!!!



        Ablauf der Installation (dokumentiert über Printscreens)
        Beschreiben Sie einzelnen Installationsschritte und Entscheidungen die Sie getroffen haben

    Test der Installation
        Dokumentieren Sie wie Sie Ihre Installation überprüfen (und was Sie dabei überprüfen)



    (Tag3) Recherche zu MySQL (Versionen, usw.)
    (Tag3) Recherche zur Installationsumgebung (OS, Kernel, etc)
    (Tag3) Dokumentation des Installationsprozesses
    (Tag3) Testing der Installation

