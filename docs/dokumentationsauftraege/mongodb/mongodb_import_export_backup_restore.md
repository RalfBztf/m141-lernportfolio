# Import, Export, Backup und Restore

## Import

### Import von JSON

Es gibt verschiedene JSON Versionen und standardmässig wird die V2.0 im relaxed mode genommen. In unserem Fall ist es eine V1.0, welche wir mit dem Parameter ```--legacy``` definieren.
Info zu den JSON V2.0: https://www.mongodb.com/docs/manual/reference/mongodb-extended-json/
Beim Connectionstring (```--uri```) wurde die DB ```cities1000``` angegeben, allerdings muss beim Import noch die DB zum importieren angegeben werden, welche ebenfalls dann ```cities1000``` heissen muss. Das ganze wird in die Collection ```cities``` gespeichert und von der Datei ```cities1000.json``` geladen.

```bash
mongoimport --uri="mongodb://localhost:27017/cities1000" --db=cities1000 --collection=cities --file=cities1000.json --legacy
```

#### Alle Städte mit der Zeitzone Europa

Zuerst DB wechseln:

```javascript
use cities1000
```

Daten suchen in Collection ```cities```:

```javascript
db.cities.find( { 
    timezone: /Europe/, 
    }, 
    { _id: 0, 
    name : 1,
    timezone: 1
    })
```

Sind relativ viele Datensätze deshalb der erste, welcher ausgegeben wurde: ```{ "name" : "Sant Julià de Lòria", "timezone" : "Europe/Andorra" }```

#### Aggregate

```javascript
db.cities.aggregate([
    {
    $match: {
        'timezone': {
        $eq: 'Europe/London'
        }
    }
    },
    {
    $group: {
        _id: 'averagePopulation',
        avgPop: {
        $avg: '$population'
        }
    }
    }
])
```

Es sucht in der Collection ```cities``` nach ```timezone``` welches genau ```Europe/London``` entspricht.
Den Output gruppiert es damit der Durchschnitt daraus erzeugt werden kann. Es ist vergleichbar mit ```GROUP BY``` in MySQL, wenn man Datensätze zählen will, wie viel ein Eintarg vor kommt.

```json
{ "_id" : "averagePopulation", "avgPop" : 23226.22149712092 }
```

#### Sort

```javascript
db.cities.aggregate([
    {
    $match: {
        'timezone': {
        $eq: 'Europe/London'
        }
    }
    },
    {
    $sort: {
        population: -1
    }
    },
    {
    $project: {
        _id: 0,
        name: 1,
        population: 1
    }
    }
])
```

Es sucht in der Collection ```cities``` nach ```timezone``` welches genau ```Europe/London``` entspricht.
Dieser Output wird absteigend sortiert. (Deshalb ```population: -1```). Es ist vergleichbar mit dem ```ORDER BY``` in MySQL, mit welchem man Daten aufsteigend oder absteigend sortieren kann. In diesem Fall werden Die Zeitzonen mit der höchsten Population zuoberst angezeigt. Ausgabe ist aber nur Stadtname und Population.

#### Weitere nützliche Vergleichsoperatoren

|Command |Description|
|:----|:----|
|\$regex |Match by any PCRE-compliant regular expression string (or just use the // delimiters as shown earlier)|
|\$ne |Not equal to|
|\$lt |Less than|
|\$lte |Less than or equal to|
|\$gt |Greater than|
|\$gte |Greater than or equal to|
|\$exists |Check for the existence of a field|
|\$all |Match all elements in an array|
|\$in |Match any elements in an array|
|\$nin |Does not match any elements in an array|
|\$elemMatch |Match all fields in an array of nested documents|
|\$or |or|
|\$nor |Not or|
|\$size |Match array of given size|
|\$mod |Modulus|
|\$type |Match if field is a given datatype|
|\$not |Negate the given operator check|

## Export mit CSV

```bash
mongoexport --db=cities1000 --collection=cities --type=csv --fields=name,timezone --out=cities1000.csv
```

### CSV wieder importieren

Zuerst alte Collection löschen:

```javascript
use cities1000
```

```javascript
db.cities.drop()
```

```bash
mongoimport --db=cities1000 --collection=cities --type=csv --columnsHaveTypes --fields="name.string(),timezone.string()" --file=cities1000.csv
```

```javascript
use cities1000
```

```javascript
db.cities.find()
```

Nun sind nur noch name und Zeitzone ersichtlich. CSV mag gut sein für Datenanalyse-Exporte, aber eher fehleranfällig und aufwendig für Restores.

## Backup (binary)

### Alle Datenbanken sichern:

```bash
mongodump --host=localhost --port=27017 --out=mongodump-2022-04-30
```

Dies erstellt eine Ordnerstruktur mit den Datenbanken und speichert die Daten im BSON-Format (Binary JSON)

### Nur die cities1000 DB:

```bash
mongodump --host=localhost --port=27017 --db=cities1000 --out=cities1000-2022-04-30
```

Dies erstellt ebenfalls eine Ordnerstruktur, aber sichert nur die ```cities1000``` DB mit also entsprechend nur einem Unterordner.

## Restore (binary)

### Datenbank cities1000 löschen:

```javascript
use cities1000
```

```javascript
db.dropDatabase()
```

### Restore:

```bash
mongorestore --uri="mongodb://localhost:27017/" cities1000-2022-04-30
```

Die Datenbank ist nun wiederhergestellt und kann verwendet werden.