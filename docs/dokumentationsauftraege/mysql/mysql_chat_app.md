# Chat App

## Einrichtung

```bash
git clone https://github.com/psachan190/realtime-chat-application-using-nodejs-expressjs-and-web-socket.git && cd realtime-chat-application-using-nodejs-expressjs-and-web-socket && npm install
```

```bash
nano database/db.js
```

Folgendes anpassen:

```js
var con = mysql.createConnection({
    host: "localhost",
    user: "chat",
    password: "Secret12?a!",
    database: "chat"
});
```

Datenbank erstellen:

```sql
CREATE DATABASE chat;
CREATE USER 'chat'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Secret12?a!';
GRANT ALL PRIVILEGES ON chat.* TO 'chat'@'localhost';
```

```bash
node server.js
```

2 Logins erstellen:

```sql
USE chat;
INSERT INTO login VALUES ('1','user1','user1');
INSERT INTO login VALUES ('2','user2','user2');
```

Ich konnte mich einloggen und das Chatten hat funktioniert.


## Umprogrammierung


### Views erstellen

```sql
CREATE VIEW v_messages AS SELECT * FROM message;
```

```sql
CREATE VIEW v_logins AS SELECT * FROM login;
```

Anzeigen mit:

```sql
show full tables where table_type = 'VIEW';
```

### Stored procedure erstellen

```sql
DELIMITER $$
CREATE PROCEDURE createMessage(IN msg VARCHAR(255), IN user VARCHAR(255))
BEGIN
	INSERT INTO message (message, user) VALUES (msg, user);
END$$
DELIMITER;
```

Anzeigen mit:

```sql
SHOW PROCEDURE STATUS WHERE db = 'chat';
```

Testen:

```sql
CALL createMessage("test","user2");
```

### Code ändern

Folgenden Code ändern

```sql
SELECT * FROM message
```

zu:

```sql
SELECT * from v_message
```

-----------------------

```sql
SELECT * FROM login WHERE username='" + username+"'
```

zu:

```sql
SELECT * FROM v_logins WHERE username='" + username +"'
```

-----------------------

```sql
INSERT INTO message (message , user) VALUES ('" + data+ "' , '"+user+"')
```

zu:

```sql
CALL createMessage('" + data + "', '"+ user + "')
```

Danach hat alles genau gleich wie vorhin funktioniert.