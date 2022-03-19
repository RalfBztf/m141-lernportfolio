# MySQL Security und Etherpad

## Etherpad Installation

### Installation von Abhängigkeiten und Applikation

Herunterladen der Repositories von NodeSources. Durch ```sudo -E bash -``` muss das Skript nicht heruntergeladen werden, sondern kann direkt ausgeführt werden.

```bash
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
```

Nun sind alle Repositories hinzugefügt und es wird Node in der Version 14 direkt von einem Node repository installiert. Wenn das Node Repository nicht hinzugefügt wird, wird bei Ubuntu 21 per Default vom Ubuntu Repository gezogen, welches aber nur die Version 12 enthält.

```bash
sudo apt install -y nodejs
```

Klonen von Etherpad aus dem Master Branch und Ordner öffnen:

```bash
git clone --branch master https://github.com/ether/etherpad-lite.git && cd etherpad-lite
```

## Etherpad Konfiguration

Kopieren des Konfigurationstemplates

```bash
cp settings.json.template settings.json
```

Anpassen der Konfigurationsdatei:

```json
"dbType" : "mysql",
"dbSettings" : {
    "user"    : "etherpad",
    "port"    : "/var/run/mysqld/mysqld.sock",
    "password": "Secret12?a",
    "database": "etherpad",
    "charset" : "utf8mb4"
},
```

Folgendes rauslöschen:

```json
  "dbType": "dirty",
  "dbSettings": {
    "filename": "var/dirty.db"
  },
```

Wir verwenden MySQL, deshalb sollte dies auch so unter ```dbType``` angegeben werden. ```user, password und database``` dürfte klar sein.
```charset``` ist einfach die Zeichenformatierung, z.B wegen Ä,Ö,Ü oder Sonderzeichen. Sehr cool ist, dass man einfach unter ```port``` den MySQL Service/Prozess angeben kann und falls der Port geändert wird, sollte Etherpad das gleich übernehmen. (Kein Hostname notwendig)

```sql
CREATE DATABASE etherpad;
CREATE USER 'etherpad'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Secret12?a';
GRANT ALL PRIVILEGES ON etherpad.* TO 'etherpad'@'localhost';
```

Hier muss ```mysql_native_password``` werden (auch gehashed), ansonsten kann die Applikation keine Verbindung herstellen.
Betreffend Security: Das MySQL-Password wird automatisch gehashed in der DB gespeichert.

## Plugins installieren

Dieser Befehl muss im obersten Verzeichnis von Etherpad ausgeführt werden:

```bash
npm install --no-save --legacy-peer-deps ep_headings2 ep_markdown ep_comments_page ep_align ep_font_color ep_webrtc ep_embedded_hyperlinks2
```

## Produktive Umgebung

Wenn authentifizierung aktiviert ist, wird empfohlen das Hash Plugin zu aktivieren, damit Passwörter nicht im Klartext rumliegen:

Authentifizierung aktivieren (wieder in settings.json):

```bash
  "requireAuthentication": true,
```

Plugin und bcrypt installieren (ebenfalls wieder im obersten Verzeichnis von Etherpad)

```bash
npm install --no-save --legacy-peer-deps ep_hash_auth bcrypt
```

Ohne bcrypt hat es bei mir nicht funktioniert. (Ist allerdings nirgends so richtig dokumentiert, würde erwarten, dass dies mit der Library mitkommt...)

Installieren des Hashgenerators (bcrypt):

```bash
sudo apt install -yqq python3-bcrypt
```

Generieren eines Hashes:

```bash
python3 -c 'import bcrypt; print(bcrypt.hashpw(b"password", bcrypt.gensalt(rounds=10, prefix=b"2a")))'
```

Kommentierung aufheben und den Hash beim User/Admin einfügen:

```json
  "users": {
    "admin": {
      // 1) "password" can be replaced with "hash" if you install ep_hash_auth
      // 2) please note that if password is null, the user will not be created
      "hash": "$2a$10$65V4MwRTKPjYwZ3mcLVw5.yGn3J.z9IFbYaXak72ucsQP9YvBOchW",
      "is_admin": true
    },
    "user": {
      // 1) "password" can be replaced with "hash" if you install ep_hash_auth
      // 2) please note that if password is null, the user will not be created
      "hash": "$2a$10$65V4MwRTKPjYwZ3mcLVw5.yGn3J.z9IFbYaXak72ucsQP9YvBOchW",
      "is_admin": false
    }
  },
```

Node produktiv machen:

```bash
export NODE_ENV=production
```

Starten der Applikation

```bash
src/bin/run.sh
```

### Daten und Testing

Ich konnte neue Pads erstellen und bearbeiten. Die ganzen Daten (teilweise JSON) werden in der Datenbank in der Tabelle ```store``` gepseichert. Es ist die einezige Tabelle. Benutzer und Settings sind alle in diesem JSON von oben gespeichert. Tabelle Store:

| Field | Type         | Null | Key | Default | Extra |
|-------|--------------|------|-----|---------|-------|
| key   | varchar(100) | NO   | PRI | NULL    |       |
| value | longtext     | NO   |     | NULL    |       |

Genauere Beschreibung (unten bei Database Structure): https://etherpad.org/doc/latest/

Der ```user``` konnte nur Pads erstellen/bearbeiten, der ```admin``` hatte zusätzlich Berechtigungen auf /admin und konnte so Plugins installieren/deinstallieren, sowie auch die Settings bearbeiten.
