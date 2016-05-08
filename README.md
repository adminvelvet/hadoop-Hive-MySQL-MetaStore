## Configuring a remote MySQL database for Hive MetaStore

By default, Hive metastore uses a Derby database, and both the database and the metastore service run embedded in the main HiveServer process. Only one process can connect to the metastore database at a time, so it is not really a practical solution but works well for unit tests.

![Hive workflow](https://github.com/gamboabdoulraoufou/hadoop-Hive/blob/master/img/hive_workflow.PNG)

In remote metastore setup, all Hive Clients will make a connection to a metastore server which in turn queries the datastore (MySQL in this example) for metadata.

![Hive workflow](https://github.com/gamboabdoulraoufou/hadoop-Hive/blob/master/img/hive_workflow.PNG)


In this post, I will show how to Configure MySQL Metastore for Hive in place of Derby Metastore

### 1- Install and start MySQL
```sh
# Install MySQL
sudo apt-get install mysql-server mysql-client python-MySQLdb

# Start MySQL
mysql --user=root --password=1234
```

### 2- Create the Database and User
```sql
/* Create database */
CREATE DATABASE metastore_db;

/* Set metastore_db as default database */
USE metastore_db;

/* Create the initial database schema using the hive-schema-0.12.0.mysql.sql */
SOURCE /home/hadoop/hive-0.12.0-bin/scripts/metastore/upgrade/mysql/hive-schema-0.12.0.mysql.sql;

/* Create user */
CREATE USER 'hive'@'%' IDENTIFIED BY '1234';

/* Set privilege for user hive */
grant all on *.* to "hive"@"%" identified by "1234";

/*  */
SHOW VARIABLES WHERE Variable_name = 'port';
SELECT user();

/* Check privileges for user hive */
show grants for 'hive'@'%';

/* Quit MySQL */
exit

```

### 3- Configure the MySQL Service and Connector
```sh
sudo apt-get install libmysql-java
ln -s /usr/share/java/mysql-connector-java.jar /home/hadoop/hive-0.12.0-bin/lib/mysql-connector-java.jar

```

### 4- Configure the metastore service to communicate with the MySQL database
```sh
# Edit or create the hive-site.xml file
sudo nano /home/hadoop/hive-0.12.0-bin/conf/hive-site.xml

# Add the content below
<configuration>
  <property>
    <name>hive.metastore.warehouse.dir</name>
    <value>/user/${user.name}/warehouse</value>
    <description>location of default database for the warehouse</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://localhost/metastore_db</value>
    <description>the URL of the MySQL database</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>hive</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>1234</value>
  </property>
  <property>
    <name>hive.metastore.schema.verification</name>
    <value>true</value>
  </property>
</configuration>

```

### 5- Check metastore configuration and restart server
```sh
schematool -dbType mysql -info
schematool -dbType mysql -initSchema

```

### 6- Restart the metastore service 
```sh
hive --service metastore

```
