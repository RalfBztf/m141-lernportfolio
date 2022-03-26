# Sakila

## Einrichtung

### SakilaDB herunterladen:

```bash
wget https://downloads.mysql.com/docs/sakila-db.zip && unzip sakila-db.zip
```

### SakilaDB importieren:

```bash
mysql -u admin01 -pSkjh9w874! < /tmp/sakila-db/sakila-schema.sql && mysql -u admin01 -pSkjh9w874! < /tmp/sakila-db/sakila-data.sql
```

### kurzes Testing

```sql
USE sakila;
SHOW FULL TABLES;
SELECT COUNT(*) FROM film;
SELECT COUNT(*) FROM film_text;
```

## Stored Procedures, Views, Triggers

### Stored Procedures

Mit Stored Procedures kann man eine oder mehrere Queries speichern und später wieder aufrufen. Dies ist hilfreich für grosse SQL Skripts, welche Sachen nacheinander und oft ausführen müssen. (z.B Daily Datenbankmutationen)

#### Erstellen

```sql
CREATE PROCEDURE procedure_name
AS
sql_statement
GO; 
```

Ausführen:

```sql
CALL procedure_name; 
```

```sql
SHOW PROCEDURE STATUS WHERE db = 'sakila';
```

| Db     | Name              | Type      | Definer           | Modified            | Created             | Security_type | Comment                                          | character_set_client | collation_connection | Database Collation |
|--------|-------------------|-----------|-------------------|---------------------|---------------------|---------------|--------------------------------------------------|----------------------|----------------------|--------------------|
| sakila | film_in_stock     | PROCEDURE | admin01@localhost | 2022-03-24 07:51:45 | 2022-03-24 07:51:45 | DEFINER       |                                                  | utf8mb4              | utf8mb4_0900_ai_ci   | utf8mb4_0900_ai_ci |
| sakila | film_not_in_stock | PROCEDURE | admin01@localhost | 2022-03-24 07:51:45 | 2022-03-24 07:51:45 | DEFINER       |                                                  | utf8mb4              | utf8mb4_0900_ai_ci   | utf8mb4_0900_ai_ci |
| sakila | rewards_report    | PROCEDURE | admin01@localhost | 2022-03-24 07:51:45 | 2022-03-24 07:51:45 | DEFINER       | Provides a customizable report on best customers | utf8mb4              | utf8mb4_0900_ai_ci   | utf8mb4_0900_ai_ci |


#### Beispiel film_in_stock

```sql
CREATE PROCEDURE film_in_stock(IN p_film_id INT, IN p_store_id INT, OUT p_film_count INT)
READS SQL DATA
BEGIN
     SELECT inventory_id
     FROM inventory
     WHERE film_id = p_film_id
     AND store_id = p_store_id
     AND inventory_in_stock(inventory_id);

     SELECT COUNT(*)
     FROM inventory
     WHERE film_id = p_film_id
     AND store_id = p_store_id
     AND inventory_in_stock(inventory_id)
     INTO p_film_count;
END $$

DELIMITER ;

DELIMITER $$
```

Die SP ```film_in_stock``` hat 3 Übergabeparameter. ```IN``` ist ein Übergabeparameter. (z.B p_store_id)  
```OUT``` heisst, dass der Wert aus der Prozedur zurückgegeben wird. Der Anfangswert ist NULL und nach der Prozedur für den Aufrufer sichtbar.

```sql
CALL film_in_stock(1,1,@count);
```

Mit ```SELECT @count``` kann nun der Counter ausgelesen werden.

```READS SQL DATA``` bedeutet, dass Daten nur gelesen werden und nicht verändert. Danach folgt die normale SQL-Squery

### Views

#### Anzeigen und erstellen

Eine View mixt Daten aus der ganzen DB zusammen, benötigt, aber nicht mehr speicher Platz, da dies normale SELECTs sind. Z.B:

```sql
CREATE VIEW v_customer AS
SELECT CustomerName, ContactName
FROM Customers
WHERE Country = "Switzerland";
```

Man könnte natürlich auch Daten aus mehreren Tabellen zusammen joinen.

Alle Views in der DB anzeigen:

```sql
show full tables where table_type = 'VIEW';
```

| Tables_in_sakila           | Table_type |
|----------------------------|------------|
| actor_info                 | VIEW       |
| customer_list              | VIEW       |
| film_list                  | VIEW       |
| nicer_but_slower_film_list | VIEW       |
| sales_by_film_category     | VIEW       |
| sales_by_store             | VIEW       |
| staff_list                 | VIEW       |

#### Beispiel actor_info

```sql
CREATE DEFINER=CURRENT_USER SQL SECURITY INVOKER VIEW actor_info
AS
SELECT
a.actor_id,
a.first_name,
a.last_name,
GROUP_CONCAT(DISTINCT CONCAT(c.name, ': ',
    (SELECT GROUP_CONCAT(f.title ORDER BY f.title SEPARATOR ', ')
                FROM sakila.film f
                INNER JOIN sakila.film_category fc
                    ON f.film_id = fc.film_id
                INNER JOIN sakila.film_actor fa
                    ON f.film_id = fa.film_id
                WHERE fc.category_id = c.category_id
                AND fa.actor_id = a.actor_id
                )
            )
            ORDER BY c.name SEPARATOR '; ')
AS film_info
FROM sakila.actor a
LEFT JOIN sakila.film_actor fa
  ON a.actor_id = fa.actor_id
LEFT JOIN sakila.film_category fc
  ON fa.film_id = fc.film_id
LEFT JOIN sakila.category c
  ON fc.category_id = c.category_id
GROUP BY a.actor_id, a.first_name, a.last_name;
```

Um die View zu nutzen braucht man die Rechte des Users bei welcher bei ```DEFINER``` angegeben ist. ```CURRENT_USER``` ist eine Variable, welche den aktuellen Benutzer ausgibt. ```SQL SECURITY INVOKER``` sagt, dass die View mit den rechten des benutzten Users ausgeführt wird. Wenn ```DEFINER``` definiert wird, wird es mit dem definierten Benutzer ausgeführt. Bei Invoker ist zu beachten, dass der Benutzer alle erforderlichen Berechtigungen auf Tabellen hat, ansonsten ist die View nicht ausführbar.


### Triggers

Mit einem Trigger kann man verursachen, dass etwas gemacht wird, sobald ein Ereignis eintritt.
Ereignisse sind:

* Eine Datenbank Manipulation (DML) wie DELETE, INSERT oder UPDATE.
* Eine Datenabnk Definition (DDL) wie CREATE, ALTER oder DROP.
* Eine Datenabnk Operation wie SERVERERROR, LOGON, LOGOFF, STARTUP, oder SHUTDOWN.

Beim Trigger muss man noch angeben, ob es vor (BEFORE), nach (AFTER) oder anstelle (INSTEAD OF) des INSERT, UPDATE oder DELETE ausgeführt wird.

Trigger erstellen:

```sql
CREATE TRIGGER customer_create_date BEFORE INSERT ON customer
FOR EACH ROW SET NEW.create_date = NOW();
```

```sql
show triggers in sakila;
```

|Trigger|Event|Table|Statement|Timing|Created|sql_mode|Definer|character_set_client|collation_connection|Database Collation|
|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|
|customer_create_date|INSERT|customer|SET NEW.create_date = NOW()|BEFORE|2022-03-24 07:51:46.96|STRICT_TRANS_TABLES,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,TRADITIONAL,NO_ENGINE_SUBSTITUTION|admin01@localhost|utf8mb4|utf8mb4_0900_ai_ci|utf8mb4_0900_ai_ci|
|ins_film|INSERT|film|BEGIN|
|    INSERT INTO film_text (film_id, title, description)|
|        VALUES (new.film_id, new.title, new.description);|
|  END|AFTER|2022-03-24 07:51:45.41|STRICT_TRANS_TABLES,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,TRADITIONAL,NO_ENGINE_SUBSTITUTION|admin01@localhost|utf8mb4|utf8mb4_0900_ai_ci|utf8mb4_0900_ai_ci|
|upd_film|UPDATE|film|BEGIN|
|    IF (old.title != new.title) OR (old.description != new.description) OR (old.film_id != new.film_id)|
|    THEN|
|        UPDATE film_text|
|            SET title=new.title,|
|                description=new.description,|
|                film_id=new.film_id|
|        WHERE film_id=old.film_id;|
|    END IF;|
|  END|AFTER|2022-03-24 07:51:45.43|STRICT_TRANS_TABLES,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,TRADITIONAL,NO_ENGINE_SUBSTITUTION|admin01@localhost|utf8mb4|utf8mb4_0900_ai_ci|utf8mb4_0900_ai_ci|
|del_film|DELETE|film|BEGIN|
|    DELETE FROM film_text WHERE film_id = old.film_id;|
|  END|AFTER|2022-03-24 07:51:45.47|STRICT_TRANS_TABLES,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,TRADITIONAL,NO_ENGINE_SUBSTITUTION|admin01@localhost|utf8mb4|utf8mb4_0900_ai_ci|utf8mb4_0900_ai_ci|
|payment_date|INSERT|payment|SET NEW.payment_date = NOW()|BEFORE|2022-03-24 07:51:47.93|STRICT_TRANS_TABLES,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,TRADITIONAL,NO_ENGINE_SUBSTITUTION|admin01@localhost|utf8mb4|utf8mb4_0900_ai_ci|utf8mb4_0900_ai_ci|
|rental_date|INSERT|rental|SET NEW.rental_date = NOW()|BEFORE|2022-03-24 07:51:48.50|STRICT_TRANS_TABLES,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,TRADITIONAL,NO_ENGINE_SUBSTITUTION|admin01@localhost|utf8mb4|utf8mb4_0900_ai_ci|utf8mb4_0900_ai_ci|

