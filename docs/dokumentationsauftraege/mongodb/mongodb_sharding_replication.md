# Sharding und Replication

## Replication

### 3 Data-Directories erstellen

```bash
mkdir ./mongo1 ./mongo2 ./mongo3
```

### Weitere MongoDB Instanzen

3 Terminals starten:

```bash
mongod --replSet book --dbpath ./mongo1 --port 27011
```

```bash
mongod --replSet book --dbpath ./mongo2 --port 27012
```

```bash
mongod --replSet book --dbpath ./mongo3 --port 27013
```

### Replikaset initialisieren

```bash
mongosh --port 27011
```

```javascript
rs.initiate({
    _id: 'book',
    members: [
      {_id: 1, host: 'localhost:27011'},
      {_id: 2, host: 'localhost:27012'},
      {_id: 3, host: 'localhost:27013'}
    ]
})
```

```json
{ ok: 1 }
```

Kontrolle:

```javascript
rs.status().ok
```

Gibt ```1``` aus, also ok.

### Verbindung zum Cluster

```bash
mongo mongodb://localhost:27011,localhost:27012,localhost:27013/replicaSet=book
```

Testdaten einfügen auf ```PRIMARY```

```javascript
db.echo.insert({ say : 'HELLO!' })
```

Primary nun abgeschaltet und auf das aktuelle Cluster verbunden:

```bash
mongo mongodb://localhost:27012,localhost:27013/replicaSet=book
```

Nach den Einträgen suchen:

```javascript
db.echo.find()
```

Einträge sind immer noch da.

```json
{ "_id" : ObjectId("628291e93a350869d435ac43"), "say" : "HELLO!" }
{ "_id" : ObjectId("6282921f3a350869d435ac44"), "say" : "HELLO1!" }
{ "_id" : ObjectId("628292493a350869d435ac45"), "say" : "HELLO!" }
{ "_id" : ObjectId("628292c93a350869d435ac46"), "say" : "HELLO!" }
```

## Sharding

### Configserver erstellen

```bash
mkdir ./mongoconfig && mongod --configsvr --replSet "config" --dbpath ./mongoconfig --port 27016
```

Login:

```bash
mongo localhost:27016
```

Cluster initialisieren:

```javascript
rs.initiate()
```

```json
{
	"info2" : "no configuration specified. Using a default configuration for the set",
	"me" : "localhost:27016",
	"ok" : 1,
	"$gleStats" : {
		"lastOpTime" : Timestamp(1652725090, 1),
		"electionId" : ObjectId("000000000000000000000000")
	},
	"lastCommittedOpTime" : Timestamp(1652725090, 1)
}
```

### Shards einrichten

```bash
mkdir ./mongo4 ./mongo5 && mongod --shardsvr --replSet "shard1" --dbpath ./mongo4 --port 27020
```

In einem anderen Terminal:

```bash
mongod --shardsvr --replSet "shard2" --dbpath ./mongo5 --port 27021
```

Shard1 initialisieren:

```bash
mongo localhost:27020
```

```javascript
rs.initiate()
```

In einem anderen Terminal Shard2 initialisieren:

```bash
mongo localhost:27021
```

```javascript
rs.initiate()
```

```json
{
	"info2" : "no configuration specified. Using a default configuration for the set",
	"me" : "localhost:27021",
	"ok" : 1
}
```

Status prüfen:

```javascript
rs.status()
```

```json
{
	"set" : "shard2",
	"date" : ISODate("2022-05-16T18:26:39.308Z"),
	"myState" : 1,
	"term" : NumberLong(1),
	"syncSourceHost" : "",
	"syncSourceId" : -1,
	"heartbeatIntervalMillis" : NumberLong(2000),
	"majorityVoteCount" : 1,
	"writeMajorityCount" : 1,
	"votingMembersCount" : 1,
	"writableVotingMembersCount" : 1,
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(1652725594, 1),
			"t" : NumberLong(1)
		},
		"lastCommittedWallTime" : ISODate("2022-05-16T18:26:34.947Z"),
		"readConcernMajorityOpTime" : {
			"ts" : Timestamp(1652725594, 1),
			"t" : NumberLong(1)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1652725594, 1),
			"t" : NumberLong(1)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1652725594, 1),
			"t" : NumberLong(1)
		},
		"lastAppliedWallTime" : ISODate("2022-05-16T18:26:34.947Z"),
		"lastDurableWallTime" : ISODate("2022-05-16T18:26:34.947Z")
	},
	"lastStableRecoveryTimestamp" : Timestamp(1652725574, 1),
	"electionCandidateMetrics" : {
		"lastElectionReason" : "electionTimeout",
		"lastElectionDate" : ISODate("2022-05-16T18:26:14.846Z"),
		"electionTerm" : NumberLong(1),
		"lastCommittedOpTimeAtElection" : {
			"ts" : Timestamp(1652725574, 1),
			"t" : NumberLong(-1)
		},
		"lastSeenOpTimeAtElection" : {
			"ts" : Timestamp(1652725574, 1),
			"t" : NumberLong(-1)
		},
		"numVotesNeeded" : 1,
		"priorityAtElection" : 1,
		"electionTimeoutMillis" : NumberLong(10000),
		"newTermStartDate" : ISODate("2022-05-16T18:26:14.893Z"),
		"wMajorityWriteAvailabilityDate" : ISODate("2022-05-16T18:26:14.914Z")
	},
	"members" : [
		{
			"_id" : 0,
			"name" : "localhost:27021",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 218,
			"optime" : {
				"ts" : Timestamp(1652725594, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2022-05-16T18:26:34Z"),
			"lastAppliedWallTime" : ISODate("2022-05-16T18:26:34.947Z"),
			"lastDurableWallTime" : ISODate("2022-05-16T18:26:34.947Z"),
			"syncSourceHost" : "",
			"syncSourceId" : -1,
			"infoMessage" : "Could not find member to sync from",
			"electionTime" : Timestamp(1652725574, 2),
			"electionDate" : ISODate("2022-05-16T18:26:14Z"),
			"configVersion" : 1,
			"configTerm" : 1,
			"self" : true,
			"lastHeartbeatMessage" : ""
		}
	],
	"ok" : 1,
	"$clusterTime" : {
		"clusterTime" : Timestamp(1652725594, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	},
	"operationTime" : Timestamp(1652725594, 1)
}
```

### Mongo-Instanz und Configserver verbinden

```bash
mongos --configdb config/localhost:27016 --port 27025
```

### Shards hinzufügen

```bash
mongo localhost:27025
```

```javascript
sh.addShard("shard1/localhost:27020")
sh.addShard("shard2/localhost:27021")
```

```json
{
	"shardAdded" : "shard1",
	"ok" : 1,
	"$clusterTime" : {
		"clusterTime" : Timestamp(1652726708, 4),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	},
	"operationTime" : Timestamp(1652726708, 4)
}
{
	"shardAdded" : "shard2",
	"ok" : 1,
	"$clusterTime" : {
		"clusterTime" : Timestamp(1652726710, 5),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	},
	"operationTime" : Timestamp(1652726710, 5)
}
```

```javascript
sh.status()
```

```json
--- Sharding Status --- 
  sharding version: {
  	"_id" : 1,
  	"minCompatibleVersion" : 5,
  	"currentVersion" : 6,
  	"clusterId" : ObjectId("628295622d516dc04b501e9a")
  }
  shards:
        {  "_id" : "shard1",  "host" : "shard1/localhost:27020",  "state" : 1,  "topologyTime" : Timestamp(1652726708, 1) }
        {  "_id" : "shard2",  "host" : "shard2/localhost:27021",  "state" : 1,  "topologyTime" : Timestamp(1652726710, 3) }
  active mongoses:
        "5.0.8" : 1
  autosplit:
        Currently enabled: yes
  balancer:
        Currently enabled: yes
        Currently running: no
        Failed balancer rounds in last 5 attempts: 0
        Migration results for the last 24 hours: 
                No recent migrations
  databases:
        {  "_id" : "config",  "primary" : "config",  "partitioned" : true }
```

Man sieht hier, MongoDB in ist die Version 5.0.8, Shard 1 läuft auf localhost:27020 und Shard 2 auf localhost:27021. Die Daten werden im Moment automatisch aufgeteilt.

### Sharding einrichten und Daten einfügen

Sharding wird auf der DB ```populations``` aktiviert.

```javascript
sh.enableSharding("populations")
```

```json
{
	"ok" : 1,
	"$clusterTime" : {
		"clusterTime" : Timestamp(1652727140, 31),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	},
	"operationTime" : Timestamp(1652727140, 30)
}
```

Die Collection ```cities``` wird nun verteilt. Mithilfe von ```hashed``` ist die Datenverteilung gleichmässiger und es ist unwahrscheinlicher, dass Dokumente mit einem ähnlichen Shard-Hashwert auf demselben Shard befinden.

```javascript
sh.shardCollection("populations.cities", { "country": "hashed" })
```

```json
{
	"collectionsharded" : "populations.cities",
	"ok" : 1,
	"$clusterTime" : {
		"clusterTime" : Timestamp(1652727173, 37),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	},
	"operationTime" : Timestamp(1652727173, 33)
}
```

DB wechseln und Daten einfügen:

```javascript
use populations
```

```javascript
db.cities.insertMany([
  {"name": "Seoul", "country": "South Korea", "continent": "Asia", "population": 25.674 },
  {"name": "Mumbai", "country": "India", "continent": "Asia", "population": 19.980 },
  {"name": "Lagos", "country": "Nigeria", "continent": "Africa", "population": 13.463 },
  {"name": "Beijing", "country": "China", "continent": "Asia", "population": 19.618 },
  {"name": "Shanghai", "country": "China", "continent": "Asia", "population": 25.582 },
  {"name": "Osaka", "country": "Japan", "continent": "Asia", "population": 19.281 },
  {"name": "Cairo", "country": "Egypt", "continent": "Africa", "population": 20.076 },
  {"name": "Tokyo", "country": "Japan", "continent": "Asia", "population": 37.400 },
  {"name": "Karachi", "country": "Pakistan", "continent": "Asia", "population": 15.400 },
  {"name": "Dhaka", "country": "Bangladesh", "continent": "Asia", "population": 19.578 },
  {"name": "Rio de Janeiro", "country": "Brazil", "continent": "South America", "population": 13.293 },
  {"name": "São Paulo", "country": "Brazil", "continent": "South America", "population": 21.650 },
  {"name": "Mexico City", "country": "Mexico", "continent": "North America", "population": 21.581 },
  {"name": "Delhi", "country": "India", "continent": "Asia", "population": 28.514 },
  {"name": "Buenos Aires", "country": "Argentina", "continent": "South America", "population": 14.967 },
  {"name": "Kolkata", "country": "India", "continent": "Asia", "population": 14.681 },
  {"name": "New York", "country": "United States", "continent": "North America", "population": 18.819 },
  {"name": "Manila", "country": "Philippines", "continent": "Asia", "population": 13.482 },
  {"name": "Chongqing", "country": "China", "continent": "Asia", "population": 14.838 },
  {"name": "Istanbul", "country": "Turkey", "continent": "Europe", "population": 14.751 }
])
```

### Testen

```javascript
db.cities.getShardDistribution()
```

```text
Shard shard2 at shard2/localhost:27021
 data : 943B docs : 9 chunks : 2
 estimated data per chunk : 471B
 estimated docs per chunk : 4

Shard shard1 at shard1/localhost:27020
 data : 1KiB docs : 11 chunks : 2
 estimated data per chunk : 567B
 estimated docs per chunk : 5

Totals
 data : 2KiB docs : 20 chunks : 4
 Shard shard2 contains 45.4% data, 45% docs in cluster, avg obj size on shard : 104B
 Shard shard1 contains 54.59% data, 55% docs in cluster, avg obj size on shard : 103B
```

Die Daten sind relativ gleichmässig verteilt.

### Anwenden

Das Routing wird nun automatisch vom Cluster übernommen, damit man beim richtigen Datensatz landet. Hier könnte man natürlich auch noch Indexe erzeugen. Im Moment werden COLLSCAN (Collection Scan) durchgeführt, was viel Leistung braucht.

```javascript
db.cities.find().explain()
```

```json
{
	"queryPlanner" : {
		"mongosPlannerVersion" : 1,
		"winningPlan" : {
			"stage" : "SHARD_MERGE",
			"shards" : [
				{
					"shardName" : "shard2",
					"connectionString" : "shard2/localhost:27021",
					"serverInfo" : {
						"host" : "m141vmclone",
						"port" : 27021,
						"version" : "5.0.8",
						"gitVersion" : "c87e1c23421bf79614baf500fda6622bd90f674e"
					},
					"namespace" : "populations.cities",
					"indexFilterSet" : false,
					"parsedQuery" : {
						
					},
					"queryHash" : "8B3D4AB8",
					"planCacheKey" : "D542626C",
					"maxIndexedOrSolutionsReached" : false,
					"maxIndexedAndSolutionsReached" : false,
					"maxScansToExplodeReached" : false,
					"winningPlan" : {
						"stage" : "SHARDING_FILTER",
						"inputStage" : {
							"stage" : "COLLSCAN",
							"direction" : "forward"
						}
					},
					"rejectedPlans" : [ ]
				},
				{
					"shardName" : "shard1",
					"connectionString" : "shard1/localhost:27020",
					"serverInfo" : {
						"host" : "m141vmclone",
						"port" : 27020,
						"version" : "5.0.8",
						"gitVersion" : "c87e1c23421bf79614baf500fda6622bd90f674e"
					},
					"namespace" : "populations.cities",
					"indexFilterSet" : false,
					"parsedQuery" : {
						
					},
					"queryHash" : "8B3D4AB8",
					"planCacheKey" : "D542626C",
					"maxIndexedOrSolutionsReached" : false,
					"maxIndexedAndSolutionsReached" : false,
					"maxScansToExplodeReached" : false,
					"winningPlan" : {
						"stage" : "SHARDING_FILTER",
						"inputStage" : {
							"stage" : "COLLSCAN",
							"direction" : "forward"
						}
					},
					"rejectedPlans" : [ ]
				}
			]
		}
	},
	"serverInfo" : {
		"host" : "m141vmclone",
		"port" : 27025,
		"version" : "5.0.8",
		"gitVersion" : "c87e1c23421bf79614baf500fda6622bd90f674e"
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
	"command" : {
		"find" : "cities",
		"filter" : {
			
		},
		"lsid" : {
			"id" : UUID("6eeb0879-96bf-45ad-88cb-9c8e51a31cf6")
		},
		"$clusterTime" : {
			"clusterTime" : Timestamp(1652727239, 32),
			"signature" : {
				"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
				"keyId" : NumberLong(0)
			}
		},
		"$db" : "populations"
	},
	"ok" : 1,
	"$clusterTime" : {
		"clusterTime" : Timestamp(1652727273, 33),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	},
	"operationTime" : Timestamp(1652727273, 33)
}
```