# MySQL Import und Export

## Import

### Import SQL-Dump

```bash
mysql -u admin01 -pSkjh9w874! pokemon < pokemon.sql
```

**Wichtig:** Falls im Dump kein ```CREATE DATABASE``` existiert, muss die Datenabnk ```pokemon``` zuerst erstellt werden.

### Import CSV-Dump

```sql
LOAD DATA INFILE '/var/lib/mysql-files/tableName.csv'
INTO TABLE dbName
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
```

Das ganze kann natürlich auch über Bash gemacht werden:

```bash
mysqlimport --ignore-lines=1 --fields-terminated-by=, dbName /var/lib/mysql-files/tableName.csv
```

```FIELDS TERMINATED BY``` ist das Trennzeichen, ```ENCLOSED BY``` wann ein Wert beginnt und aufhört (z.B "Tier"), ```LINES TERMINATED BY``` zeigt auf, wann ein Datensatz zu ende ist, ```IGNORE 1 ROWS``` ignoriert die erste Zeile, da dies oft der Header mit den Attributennamen ist.

## Export

### Export SQL-Dump

#### Einzelne Tabellen dumpen

Hier werden die Tabellen ```pokemon``` und ```top_10_view``` der DB ```pokemon``` exportiert.

```bash
mysqldump -u admin01 -pSkjh9w874! pokemon pokemon top_10_view > dump.sql
```

Falls die ganze Datenbank mit allen Tabellen exportiert werden will:

```bash
mysqldump -u admin01 -pSkjh9w874! pokemon > dump.sql
```

#### Mehrere Datenbanken

```bash
mysqldump -u admin01 -pSkjh9w874! --databases pokemon, schooltest > dump.sql
```

#### Alle Datenbanken

```bash
mysqldump -u admin01 -pSkjh9w874! --all-databases > dump.sql
```

### Export CSV-Dump

Export als CSV mit Pokemons, die einen "fire-punch" als Move durchführen können:

```sql
select pokemon.pok_name from pokemon inner join pokemon_moves ON pokemon.pok_id = pokemon_moves.pok_id where pokemon_moves.move_id = 7 INTO OUTFILE '/var/lib/mysql-files/pokemon.csv' FIELDS ENCLOSED BY '"' TERMINATED BY ';' LINES TERMINATED BY '\n';
```
