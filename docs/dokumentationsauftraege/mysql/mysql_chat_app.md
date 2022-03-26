# Chat-App

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


## Umprogrammierung
