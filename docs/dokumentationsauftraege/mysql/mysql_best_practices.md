# Best practices

## Kurze Zusammenfassung

Bei Update Statements nutzt man oft ```WHERE``` (z.B ```UPDATE .... WHERE userId = 1```) und da macht MySQL einen Full-Table-Scan, wenn userId nicht indexiert ist. (langsam)
Wenn jedoch userId indexiert ist, weiss es genau wo das Element liegt (schnell). Indexieren sollte man zusätzlich aber auch Attribute, welche in ```ORDER BY ``` oder ```GROUP BY ``` verwendet werden.

Bei ```WHERE``` statements ist es schlecht viele ```OR``` zu verwenden. Besser ist mehrere SELECTs zu machen und diese mit ```UNION``` zu verbinden. Ansonste könnte es sein, dass trotzdem ein Full-Table-Scan ausgeführt wird.

Beispiel:

```sql
SELECT * FROM students WHERE first_name LIKE 'Ade%' UNION ALL SELECT * FROM students WHERE last_name LIKE 'Ade%' ;
```

Falls anstelle von ```UNION ALL``` nur ```UNION``` verwendet wird, werden einfach noch die Duplikate entfernt.

Diese Wildcards von oben sind aber trotzdem noch nicht so gut. (```... WHERE first_name LIKE '%Ade';```)
Da ist es besser eine Volltextsuche für diese Felder hinzuzufügen:

```sql
ALTER TABLE students ADD FULLTEXT (first_name, last_name);
```

Nun können wir einfach per Volltextsuche nach ```Ade``` in den Feldern ```first_name``` und ```last_name``` suchen:

```sql
SELECT * FROM students WHERE match(first_name, last_name) AGAINST ('Ade');
```

Auch beim Design der DB kann bereits darauf achten Sachen zu optimieren:

* Tabellen normalisieren / keine redudanten Daten
* Geschickte Datentypen verwenden
* NULL-Werte vermeiden
* Zu viele Spalten(Attribute) im Design vermeiden
* Bei SQL-Joins vorsichtig sein (nur die Tabellen einbeziehen, die auch nötig sind) - nur Daten ausgeben die nötig sind (kein "Select *")

MySQL kann auch Querys cachen. Dazu gibt es Variablen, welche die Settings anzeigen:

```sql
SHOW VARIABLES LIKE 'query_cache_%';
```

|Variable_name|Value|
|:----|:----|
|query_cache_limit|1048576|
|query_cache_min_res_unit|4096|
|query_cache_size|16777216|
|query_cache_type|OFF|
|query_cache_wlock_invalidate|OFF|

Kann in ```/etc/mysql/my.cnf``` angepasst werden. z.B

```bash
query_cache_type=1          # 1 ist eingestellt / 0 ist aus
query_cache_size = 10M      # default ist 1MB / erhöhen auf 10MB macht Sinn
query_cache_limit=1M        # Individuelle Query-Resultate die gecacht werden, 1MB macht Sinn 
```

## Übung Performance und Indexing

Einlesen der DB:

```sql
sudo mysql < classicModelohneIndexv2.0.sql
```

## Joins ohne Indexe

```sql
EXPLAIN SELECT * FROM orderdetails d     
    INNER JOIN orders o ON d.orderNumber = o.orderNumber    
    INNER JOIN products p ON p.productCode = d.productCode     
    INNER JOIN productlines l ON p.productLine = l.productLine     
    INNER JOIN customers c on c.customerNumber = o.customerNumber      
WHERE o.orderNumber = "10101";
```

|id|select_type|table|partitions|type|possible_keys|key|key_len|ref|rows|filtered|Extra|
|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|
|1|SIMPLE|l|NULL|ALL|NULL|NULL|NULL|NULL|7|100.00|NULL|
|1|SIMPLE|p|NULL|ALL|NULL|NULL|NULL|NULL|110|10.00|Using where; Using join buffer (hash join)|
|1|SIMPLE|o|NULL|ALL|NULL|NULL|NULL|NULL|326|10.00|Using where; Using join buffer (hash join)|
|1|SIMPLE|c|NULL|ALL|NULL|NULL|NULL|NULL|122|10.00|Using where; Using join buffer (hash join)|
|1|SIMPLE|d|NULL|ALL|NULL|NULL|NULL|NULL|2996|1.00|Using where; Using join buffer (hash join)|

Wären gerechnet 7 x 110 x 326 x 122 x 2996 = 91750822240 gescannte Elemente

```type``` Sagt aus, wie die Daten zusammengejoint werden. ALL ist aber sehr schlecht.
```possible_keys``` Sagt aus, welche (Index)Schlüssel verwendet werden könnten. NULL

Dazu die Tabelle frisieren:

```sql
ALTER TABLE customers ADD PRIMARY KEY (customerNumber);
ALTER TABLE employees ADD PRIMARY KEY (employeeNumber);
ALTER TABLE offices ADD PRIMARY KEY (officeCode);
ALTER TABLE orderdetails ADD PRIMARY KEY (orderNumber, productCode);
ALTER TABLE orders ADD PRIMARY KEY (orderNumber), ADD KEY (customerNumber);
ALTER TABLE payments ADD PRIMARY KEY (customerNumber, checkNumber);
ALTER TABLE productlines ADD PRIMARY KEY (productLine);
ALTER TABLE products ADD PRIMARY KEY (productCode), ADD KEY (buyPrice), ADD KEY (productLine);
ALTER TABLE productvariants ADD PRIMARY KEY (variantId),ADD KEY (buyPrice),ADD KEY (productCode);
```

Danach nochmals den Explain ausführen:

```sql
EXPLAIN SELECT * FROM orderdetails d     
    INNER JOIN orders o ON d.orderNumber = o.orderNumber    
    INNER JOIN products p ON p.productCode = d.productCode     
    INNER JOIN productlines l ON p.productLine = l.productLine     
    INNER JOIN customers c on c.customerNumber = o.customerNumber      
WHERE o.orderNumber = "10101";
```

und wir braüchten nur noch  1 x 1 x 4 x 1 x 1 = 4 Elemente zu scannen

|id|select_type|table|partitions|type|possible_keys|key|key_len|ref|rows|filtered|Extra|
|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|
|1|SIMPLE|o|NULL|const|PRIMARY,customerNumber|PRIMARY|4|const|1|100.00|NULL|
|1|SIMPLE|c|NULL|const|PRIMARY|PRIMARY|4|const|1|100.00|NULL|
|1|SIMPLE|d|NULL|ref|PRIMARY|PRIMARY|4|const|4|100.00|Using index condition|
|1|SIMPLE|p|NULL|eq_ref|PRIMARY,productLine|PRIMARY|17|explaintesting.d.productCode|1|100.00|NULL|
|1|SIMPLE|l|NULL|eq_ref|PRIMARY|PRIMARY|52|explaintesting.p.productLine|1|100.00|NULL|