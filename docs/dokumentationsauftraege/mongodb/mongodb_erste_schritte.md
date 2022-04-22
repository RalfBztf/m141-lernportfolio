# Erste Schritte

## Mongo-Shell starten

```bash
mongo --host localhost --port 27017
```

## DB erstellen

Zuerst brauchen wir eine Datenbank

```javascript
use testdb
```

Mit ```use``` wird die Datenbank automatisch erstellt. Folgende Meldung sollte erscheinen:

```output
switched to db testdb
```

## Collection erstellen

```javascript
db.createCollection ("testcollection", { capped: true, size: 6142800, max: 10000 })
```

Nun ist die grösse der Collection auf 6'142'800 Bytes und max 10'000 Documents beschränkt.
Erfolgreich erstellt ist sie wenn ```{ "ok" : 1 }``` zurückgegeben wird.

## Documents erstellen, suchen und löschen

### Erstellen

```javascript
db.testcollection.insertOne({Name: "Waelchli", Klasse: "IN19-23a"})
```

Wenn mehrere oder verschiedene Documents erstellt werden wollen, kann anstelle von ```insertOne``` auch ```insertMany``` oder ```insert``` verwendet werden.

Folgender output sollte ersichtlich sein, wenn das Document erfolgreich erstellt wurde:

```json
{
    "acknowledged" : true,
    "insertedId" : ObjectId("6262f4f9bd2a793e86054a17")
}
```

Die ```ObjectId``` ist ein Primary-Key für diesen Eintrag.

### Suchen

```javascript
db.testcollection.find( { Name: "Waelchli"} )
```

Eintrag mit Name ```Waelchli``` wurde gefunden:

```json
{ "_id" : ObjectId("6262f4f9bd2a793e86054a17"), "Name" : "Waelchli", "Klasse" : "IN19-23a" }
```

### Bearbeiten

Bei ```Waelchli``` die Klasse auf ```IN19-23d``` setzen:

```javascript
db.testcollection.update({ Name: "Waelchli" },{$set: { Klasse: "IN19-23d" }})
```

Update besteht aus 2 Objekten, das Suchobjekt ```Ẁaelchli``` und das Objekt mit den neuen Daten.

```json
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```

Bei einer neuen Suche:

```json
{ "_id" : ObjectId("6262f4f9bd2a793e86054a17"), "Name" : "Waelchli", "Klasse" : "IN19-23d" }
```


### Löschen

```javascript
db.testcollection.remove ( { Name: "Waelchli" } )
```

Wenn erfolgreich gelöscht:

```json
WriteResult({
    "nRemoved" : 0,
    "writeError" : {
        "code" : 136,
        "errmsg" : "CollectionScan died due to position in capped collection being deleted. Last seen record id: RecordId(1)"
    }
})
```

Oder nur den ersten zutreffenden Eintrag löschen

```javascript
db.testcollection.remove ( { Name: "Waelchli" }, 1 )
```

## GUI-Client

Installieren von MongoDB Compass:

```bash
wget https://downloads.mongodb.com/compass/mongodb-compass_1.31.1_amd64.deb && sudo dpkg -i mongodb-compass_1.31.1_amd64.deb
```

```bash
mongodb-compass
```
Oder suchen über Startmenu.

Hier kann man sich sehr einfach auf einen Server verbinden, deren Collections anschauen und auch direkt in die Dokumente sehen bzw. Daten anpassen.