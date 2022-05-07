# Benutzer und URI

## Vorbereitungen

Repository klonen und in Ordner navigieren:

```bash
git clone https://github.com/sbhaseen/express-dashboard-demo.git && cd express-dashboard-demo
```

Abhängigkeiten installieren und Applikation im Development-Modus starten:

```bash
npm install && npm run dev
```

Unter http://localhost:5000 ist die Applikation nun erreichbar.

## Testen

Ich habe zum testen neue Maschinen, Kategorien und Prozesse angelegt. Diese haben alle Ihren eigenen Controller. ```machineController.js```, ```categoryController.js``` und ```processController.js```. Die Controller rufen wiederum die Models auf. ```category.js```, ```machine.js``` und ```process.js```.

Um die Daten in der DB anzuschauen (DB in server.js angegeben):

```javascript
use mfgdashboard;
```

### Collections anzeigen:

```javascript
show collections;
```

### Dokumente inspizieren

```javascript
db.machines.find();
```

```json
{ "_id" : ObjectId("62736eb9ade60e3acc1e4599"), "name" : "Testmaschine", "date_of_commissioning" : ISODate("2022-03-03T00:00:00Z"), "date_of_retirement" : ISODate("2022-05-09T00:00:00Z"), "__v" : 0 }
```

```javascript
db.categories.find();
```

```json
{ "_id" : ObjectId("62736ecdade60e3acc1e459a"), "name" : "Testcategory", "__v" : 0 }
```

```javascript
db.processes.find();
```

```json
{ "_id" : ObjectId("62736ee1ade60e3acc1e459b"), "category" : [ ObjectId("62736ecdade60e3acc1e459a") ], "name" : "Testprocess", "machine" : ObjectId("62736eb9ade60e3acc1e4599"), "summary" : "Test1", "serial_number" : "0473", "__v" : 0 }
```

## Erweiterung

### MongoDB Authentifizierung aktivieren

```bash
sudo nano /etc/mongod.conf
```

Folgendes hinzufügen:

```conf
security:
    authorization: enabled
```

Server neustarten:

```bash
sudo systemctl restart mongod.service
```

Mit der Authentifizierung wird gleich auch Authorisierung aktiviert (RBAC). Mit Role-Based-Access-Control werden Rollen mit Rechten den Benutzern zugeordnet und nicht wie in MySQL direkt ein Recht. Es gibt grundsätzlich 2 Arten von Rollen. Built-in-Rollen und Self-Signed-Rollen.
Einer Rolle werden Priviliegien zugeordnet, welche bestimmt, welche Aktionen (z.B insert, update ...) man auf welche Ressourcen (z.B DBs oder Collections) ausführen darf.
* Actions: https://www.mongodb.com/docs/manual/reference/privilege-actions/

### MongoDB Systemadmin erstellen

Auf ```admin``` DB wechseln:

```javascript
use admin
```

```javascript
db.createUser(
  {
    user: "sysadmin",
    pwd: "test1234",
    roles: [ { role: "root", db: "admin" } ]
  }
)
```

```json
Successfully added user: {
    "user" : "sysadmin",
    "roles" : [
        {
            "role" : "root",
            "db" : "admin"
        }
    ]
}
```

### Applikationsbenutzer erfassen

Auf ```mfgdashboard``` DB wechseln:

```javascript
use mfgdashboard
```

Rolle mit den Rechten ```find, update insert, remove``` in der DB ```mfgdashboard``` erstellen:

```javascript
db.createRole(
   {
     role: "mfgdashboardRole",
     privileges: [
       { resource: { db: "mfgdashboard", collection: "" }, actions: ["find", "update", "insert", "remove"] }
     ],
     roles: []
   }
)
```

Benutzer erfassen mit der ```mfgdashboardRole``` Rolle:

```javascript
db.createUser(
  {
    user: "expressdemo",
    pwd: "test1234",
    roles: [ { role: "mfgdashboardRole", db: "mfgdashboard" } ]
  }
)
```

Falls die Rolle nicht mehr gebraucht wird:

```javascript
db.runCommand(
    { revokeRolesFromUser: "expressdemo",
        roles: ["mfgdashboardRole"]
    }
)
```

Login mit Benutzer:

```bash
mongo -p 27017 --authenticationDatabase "mfgdashboard" -u expressdemo -p
```

Versuchen eine Collection zu droppen:

```javascript
use mfgdashboard
```

```javascript
db.machines.drop()
```

```json
uncaught exception: Error: drop failed: {
	"ok" : 0,
	"errmsg" : "not authorized on mfgdashboard to execute command { drop: \"machines\", lsid: { id: UUID(\"847d813b-6816-4488-be59-7cf5e11c0a3f\") }, $db: \"mfgdashboard\" }",
	"code" : 13,
	"codeName" : "Unauthorized"
} :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
DBCollection.prototype.drop@src/mongo/shell/collection.js:713:15
@(shell):1:1
```

### Server.js anpassen

```bash
nano server.js
```

```javascript
const db = process.env.MONGODB_URI || 'mongodb://expressdemo:test1234@localhost:27017/mfgdashboard';
```

Das Passwort sollte man in einer Prod-Umgebung nicht einfach so in die JS-Datei schreiben, sondern besser den Connection String als Umgebungsvariable definieren.

Applikation starten

```bash
npm run dev
```

Durch hinzufügen von Maschinen, Prozessen etc. stürzte die Applikation nicht ab, also sollten die 4 Rechte der Rolle vorerst ausreichen.

