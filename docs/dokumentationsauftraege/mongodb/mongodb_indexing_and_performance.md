# Indexing und Performance

## Vorbereitung

### DB erstellen

```javascript
use phones
```

### Datengenerierungsfunktion erstellen

```javascript
populatePhones = function(area, start, stop) {
      for(var i = start; i < stop; i++) {
        var country = 1 + ((Math.random() * 8) << 0);
        var num = (country * 1e10) + (area * 1e7) + i;
        var fullNumber = "+" + country + " " + area + "-" + i;
        db.phones.insert({
          _id: num,
          components: {
            country: country,
            area: area,
            prefix: (i * 1e-4) << 0,
            number: i
          },
          display: fullNumber
        });
        print("Inserted number " + fullNumber);
      }
      print("Done!");
    }
```

### Daten generieren

```javascript
populatePhones(800, 5550000, 5650000)
```

### 5 Datensätze anzeigen

```javascript
db.phones.find().limit(5)
```

```json
{ "_id" : 68005550000, "components" : { "country" : 6, "area" : 800, "prefix" : 555, "number" : 5550000 }, "display" : "+6 800-5550000" }
{ "_id" : 18005550001, "components" : { "country" : 1, "area" : 800, "prefix" : 555, "number" : 5550001 }, "display" : "+1 800-5550001" }
{ "_id" : 78005550002, "components" : { "country" : 7, "area" : 800, "prefix" : 555, "number" : 5550002 }, "display" : "+7 800-5550002" }
{ "_id" : 38005550003, "components" : { "country" : 3, "area" : 800, "prefix" : 555, "number" : 5550003 }, "display" : "+3 800-5550003" }
{ "_id" : 38005550004, "components" : { "country" : 3, "area" : 800, "prefix" : 555, "number" : 5550004 }, "display" : "+3 800-5550004" }
```

## Indices auslesen

```javascript
db.getCollectionNames().forEach(function(collection) {
     print("Indexes for the " + collection + " collection:");
     printjson(db[collection].getIndexes());
});
```

```json
[ { "v" : 2, "key" : { "_id" : 1 }, "name" : "_id_" } ]
```

Es gibt keine Indexe ausser die ObjectID von MongoDB.

## Performance ohne Indices

```javascript
db.phones.explain("executionStats").find({display: "+1 800-5650001"})
```

```json
{
	"explainVersion" : "1",
	"queryPlanner" : {
		"namespace" : "phones.phones",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"display" : {
				"$eq" : "+1 800-5650001"
			}
		},
		"maxIndexedOrSolutionsReached" : false,
		"maxIndexedAndSolutionsReached" : false,
		"maxScansToExplodeReached" : false,
		"winningPlan" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"display" : {
					"$eq" : "+1 800-5650001"
				}
			},
			"direction" : "forward"
		},
		"rejectedPlans" : [ ]
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 0,
		"executionTimeMillis" : 82,
		"totalKeysExamined" : 0,
		"totalDocsExamined" : 100000,
		"executionStages" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"display" : {
					"$eq" : "+1 800-5650001"
				}
			},
			"nReturned" : 0,
			"executionTimeMillisEstimate" : 13,
			"works" : 100002,
			"advanced" : 0,
			"needTime" : 100001,
			"needYield" : 0,
			"saveState" : 100,
			"restoreState" : 100,
			"isEOF" : 1,
			"direction" : "forward",
			"docsExamined" : 100000
		}
	},
	"command" : {
		"find" : "phones",
		"filter" : {
			"display" : "+1 800-5650001"
		},
		"$db" : "phones"
	},
	"serverInfo" : {
		"host" : "m141vm",
		"port" : 27017,
		"version" : "5.0.7",
		"gitVersion" : "b977129dc70eed766cbee7e412d901ee213acbda"
	},
	"serverParameters" : {
		"internalQueryFacetBufferSizeBytes" : 104857600,
		"internalQueryFacetMaxOutputDocSizeBytes" : 104857600,
		"internalLookupStageIntermediateDocumentMaxSizeBytes" : 104857600,
		"internalDocumentSourceGroupMaxMemoryBytes" : 104857600,
		"internalQueryMaxBlockingSortMemoryUsageBytes" : 104857600,
		"internalQueryProhibitBlockingMergeOnMongoS" : 0,
		"internalQueryMaxAddToSetBytes" : 104857600,
		"internalDocumentSourceSetWindowFieldsMaxMemoryBytes" : 104857600
	},
	"ok" : 1
}
```

```"stage": "COLLSCAN"``` bedeutet, dass ein Scan auf die Collections durchgeführt wurde und nicht auf z.B die Indexe (```IXSCAN```), dies braucht viel Rechenpower und dauerte in meinem Fall: 82ms ```"executionTimeMillis" : 82``` für 100000 durchsuchte Dokumente ```"totalDocsExamined" : 100000```

[Weitere Erklärungen für Explains](https://www.mongodb.com/docs/upcoming/reference/explain-results/)

## Neuen Index erfassen

```javascript
db.phones.createIndex(
  { display : 1 },
  { unique : true, dropDups : true }
)
```

```json
{
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"createdCollectionAutomatically" : false,
	"ok" : 1
}
```

Nochmals Indices auslesen:

```javascript
db.getCollectionNames().forEach(function(collection) {
     print("Indexes for the " + collection + " collection:");
     printjson(db[collection].getIndexes());
});
```

```json
[
	{
		"v" : 2,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_"
	},
	{
		"v" : 2,
		"key" : {
			"display" : 1
		},
		"name" : "display_1",
		"unique" : true
	}
]
```

Nun gibt es auch einen Index für ```display```.

Nochmals Performance testen:

```javascript
db.phones.explain("executionStats").find({display: "+1 800-5650001"})
```

```json
{
	"explainVersion" : "1",
	"queryPlanner" : {
		"namespace" : "phones.phones",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"display" : {
				"$eq" : "+1 800-5650001"
			}
		},
		"maxIndexedOrSolutionsReached" : false,
		"maxIndexedAndSolutionsReached" : false,
		"maxScansToExplodeReached" : false,
		"winningPlan" : {
			"stage" : "FETCH",
			"inputStage" : {
				"stage" : "IXSCAN",
				"keyPattern" : {
					"display" : 1
				},
				"indexName" : "display_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"display" : [ ]
				},
				"isUnique" : true,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "forward",
				"indexBounds" : {
					"display" : [
						"[\"+1 800-5650001\", \"+1 800-5650001\"]"
					]
				}
			}
		},
		"rejectedPlans" : [ ]
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 0,
		"executionTimeMillis" : 1,
		"totalKeysExamined" : 0,
		"totalDocsExamined" : 0,
		"executionStages" : {
			"stage" : "FETCH",
			"nReturned" : 0,
			"executionTimeMillisEstimate" : 0,
			"works" : 1,
			"advanced" : 0,
			"needTime" : 0,
			"needYield" : 0,
			"saveState" : 0,
			"restoreState" : 0,
			"isEOF" : 1,
			"docsExamined" : 0,
			"alreadyHasObj" : 0,
			"inputStage" : {
				"stage" : "IXSCAN",
				"nReturned" : 0,
				"executionTimeMillisEstimate" : 0,
				"works" : 1,
				"advanced" : 0,
				"needTime" : 0,
				"needYield" : 0,
				"saveState" : 0,
				"restoreState" : 0,
				"isEOF" : 1,
				"keyPattern" : {
					"display" : 1
				},
				"indexName" : "display_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"display" : [ ]
				},
				"isUnique" : true,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "forward",
				"indexBounds" : {
					"display" : [
						"[\"+1 800-5650001\", \"+1 800-5650001\"]"
					]
				},
				"keysExamined" : 0,
				"seeks" : 1,
				"dupsTested" : 0,
				"dupsDropped" : 0
			}
		}
	},
	"command" : {
		"find" : "phones",
		"filter" : {
			"display" : "+1 800-5650001"
		},
		"$db" : "phones"
	},
	"serverInfo" : {
		"host" : "m141vm",
		"port" : 27017,
		"version" : "5.0.7",
		"gitVersion" : "b977129dc70eed766cbee7e412d901ee213acbda"
	},
	"serverParameters" : {
		"internalQueryFacetBufferSizeBytes" : 104857600,
		"internalQueryFacetMaxOutputDocSizeBytes" : 104857600,
		"internalLookupStageIntermediateDocumentMaxSizeBytes" : 104857600,
		"internalDocumentSourceGroupMaxMemoryBytes" : 104857600,
		"internalQueryMaxBlockingSortMemoryUsageBytes" : 104857600,
		"internalQueryProhibitBlockingMergeOnMongoS" : 0,
		"internalQueryMaxAddToSetBytes" : 104857600,
		"internalDocumentSourceSetWindowFieldsMaxMemoryBytes" : 104857600
	},
	"ok" : 1
}
```

Nun wurde der Index (```IXSCAN```) ausgeführt innert einer Millisekunde und musste auch keine Dokumente mehr durchsuchen, da alles indexiert ist.