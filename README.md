# Configuring a remote MySQL database for Hive MetaStore

By default, Hive metastore uses a Derby database, and both the database and the metastore service run embedded in the main HiveServer process. Only one process can connect to the metastore database at a time, so it is not really a practical solution but works well for unit tests.

![Hive workflow](https://github.com/gamboabdoulraoufou/hadoop-Hive/blob/master/img/hive_workflow.PNG)

In remote metastore setup, all Hive Clients will make a connection to a metastore server which in turn queries the datastore (MySQL in this example) for metadata.

![Hive workflow](https://github.com/gamboabdoulraoufou/hadoop-Hive/blob/master/img/hive_workflow.PNG)


In this post, I will show how to Configure MySQL Metastore for Hive in place of Derby Metastore

### 5- Hive trouble shooting
```sh
schematool -dbType derby -initSchema
```

# Install and start MySQL
```sh
# Install MySQL
sudo apt-get install mysql-server mysql-client python-MySQLdb

# Start MySQL
mysql --user=root --password=1234
```

# Create the Database and User
```sql
CREATE DATABASE metastore_db;
USE metastore;
CREATE USER 'hive'@'localhost' IDENTIFIED BY '1234';
REVOKE ALL PRIVILEGES, GRANT OPTION FROM 'hive'@'localhost';
GRANT ALL PRIVILEGES ON metastore_db.* TO 'hive'@'localhost';
FLUSH PRIVILEGES;
SHOW VARIABLES WHERE Variable_name = 'port';
SELECT user();
SOURCE /home/hadoop/hive-0.12.0-bin/scripts/metastore/upgrade/mysql/hive-schema-0.12.0.mysql.sql;
exit
```

# Configure the MySQL Service and Connector
```sh
sudo apt-get install libmysql-java
ln -s /usr/share/java/mysql-connector-java.jar /home/hadoop/hive-0.12.0-bin/lib/mysql-connector-java.jar
```sh

# check metastore configuration
```sh
schematool -dbType mysql -initSchema
schematool -dbType mysql -info
```

# Configure the metastore service to communicate with the MySQL database


```xml
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

```

# 
```sh
hive --service metastore
```
grant all on *.* to "hive"@"%" identified by "1234";
