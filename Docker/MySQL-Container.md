# 💽MySQL Container
---
## ⚠️ Requirements
- Docker
---

### 1: Downloading the container
First of all we need to run this command

```docker run -d -p 33060:3306 --name mysql-db -e MYSQL_ROOT_PASSWORD=secret mysql```

- **d**:  Detached mode, the way to run in the background.

- **p** : Port, the container runs on port 3306 but we bind it so we can listen on port 33061 of the host.

- **name** : To avoid having to reference the hash, we assign it a name.

- **e** : Environment, we assign the password.

### 2: Run the container
Once we have our container created we need to start it:

```docker start mysql-db```

### 3: Connecting to the database
This are the relevant inforamtion you need to connect to this database:

- **Connection Type**: "Standard TCP/IP
- **Host**: localhost
- **Port**: 33060
- **Username**: root
- **Password**: secret

