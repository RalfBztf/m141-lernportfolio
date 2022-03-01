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

#### PERFORMANCE_SCHEMA

### Anwendung einer Engine

Die Anwendung einer Engine auf eine Tabele funktioniert wie folgt:

## Benutzer und Berechtigungen

### Demo DB

MySQL-Shell starten:

```bash
sudo mysql
```

```sql
CREATE DATABASE demoDB;
CREATE TABLE()
```



    Storage Engines bei MySQL
        Recherche - Dokumentieren Sie ausführlich die 2 wichtigsten (nicht irgendwelche, die wichtigsten) Storage-Engines unter MySQL
        Recherche - Listen und beschreiben Sie kurz die restlichen Storage-Engines auf Ihrem System
        Überlegen Sie sich welche Daten mit welcher Storage Engine gespeichert werden könnten
        Dokumentieren Sie wie eine Storage Engine auf eine Tabelle angewendet wird

    Benutzer und Berechtigungen
        Erstellen Sie eine Demo-Datenbank mit zwei Tabellen
        Root-Benutzer konfiguriert (Login/Passwort)
        Benutzer konfiguriert (Login/Passwort/Berechtigung auf Datenbank eingeschränkt <- ein Anwendungsbenutzer)
        Admin-Benutzer konfiguriert (Login/Passwort/Berechtigung auf Datenbank eingeschränkt)
        Verfizieren Sie Ihre Konfiguration und speichern Sie das Resultat in Ihrer Dokumentation

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
