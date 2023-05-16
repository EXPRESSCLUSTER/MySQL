MySQL Database 8.0.33 Quick Start Guide for EXPRESSCLUSTER X on Windows
===

About this guide
---
This guide describes how to setup MySQL with EXPRESSCLUSTER X 5.1.

For the detailed information of EXPRESSCLUSTER X, please refer to [this site](https://www.nec.com/en/global/prod/expresscluster/index.html) .


## **System Overview**

### **System Requirement**
- Using 2 servers
  - IP reachable to each other
  - Having mirror disk
    - At least 2 partitions are required on each mirror disk.
    - Cluster partition size depends on ECX version.
    - X 5.1 or later: 1024MB
    - Data partition size depends on Database sizing.
- MySql Database are installed on local partition on each server.

### **System Configuration**
- Windows Server 2022 Standard
- MySQL 8.0.33
- EXPRESSCLUSTER X 5.1

        Sample configuration
		<LAN>
		 |  +--------------------------+
		 |  | Primary Server           |
		 |  | - Wincluster1 (Win2k22)  |
		 |  | - MySQL 8.0.33           |
		 |  | - EXPRESSCLUSTER X 5.1   |
		 |  | IP Address:10.0.7.60     |
		 |  | RAM   : 6GB              |
		 |  | Disk 0: 50GB OS          |
		 |  |      C: local partition  |
		 |  | Disk 1: 10GB mirror disk |
		 |  |      E: cluster partition|
		 |  |      D: mirror partition |
		 |  +-----------+--------------+
		 |              |
		 |              | Mirroring
		 |              |
		 |  +-----------+--------------+
		 |  | Secondary Server         |
		 |  | - Wincluster2 (Win2k22)  |
		 |  | - MySQL 8.0.33           |
		 |  | - EXPRESSCLUSTER X 5.1   |
		 |  | IP Address:10.0.7.63     |
         |  | RAM   : 6GB              |
		 |  | Disk 0: 50GB OS          |
		 |  |      C: local partition  |
		 |  | Disk 1: 10GB mirror disk |
		 |  |      E: cluster partition|
		 |  |      D: mirror partition |
		 |  +--------------------------+
		 |

#### Cluster configuration
- Network Partition Resolution resource (PING method) : `pingnp1`

- failover Group: `Mysqlfailover`
	- Group resource
		- Fip                : Floating IP address resource
		- md                 : mirror disk resource
		- MySQL_service      : service resource for MySQL80

- Monitor resource

	- Fipw1             : Floating IP monitor resource
	- mdw1              : mirror disk monitor resource
	- servicew1         : service monitor resource for Mysql_service
	- userw             : user-mode monitor resource


## **Basic cluster setup on Primary and Secondary servers**


 ### **1. Install EXPRESSCLUSTER X (ECX)**
 ### **2. Register ECX licenses**
- EXPRESSCLUSTER X 5.1 for Windows
- EXPRESSCLUSTER X Replicator 5.1 for Windows

### **3. Create a cluster and a failover group On Primary server**
    - Network partition: 
        pingnp1: PING method
    - Group:
        Mysqlfailover
        Fip: Floating IP resource
        md : mirror disk resource
        
### **4. Start group on Primary server**

      
       +----------------------+-----------------------------+
       | Parameter            | Value                       |
       +----------------------+-----------------------------+
       | Name                 | SqlFailover                 |
       | Startup Server       | wincluster1 -> wincluster2  |
       | Floating IP Address  | 10.0.7.162                  |
       | Mirror Disk resource | D:\Mysql                    |
       +----------------------+-----------------------------+

 If want to know how to add the resource, please refer to "EXPRESSCLUSTER X 5.1 for Windows Installation and Configuration Guide". 

 After you add failver group and execute apply the configuration file, you start failover group by wincluster1.  
     
### **5. Install MySQL on both servers**

- Install MySQL 8.0.33 on both the servers.
    
    - For installation procedure please refer to [this site](https://dev.mysql.com/doc/mysql-installation-excerpt/8.0/en/ ).

### **6. MySQL Configuration for Mirror disk (Node1)**

- Create the database directory.
- Create data directory in mirror drive e.g. D:\MySQL
- Stop mysql service from services.msc.
- Copy MySQL data from default directory (C:\ProgramData\MySQL\MySQL Server 8.0\MySQL) to Mirror disk (D:\MySQL)
      

### **7. Perform below steps on both the Nodes one by one after group failover move.**

 - Configure the MySQL Configuration file (C:\ProgramData\MySQL\MySQL Server 8.0\my.ini).
    > [mysqld]  
    > datadir=D:\MySQL 
    

 
### **8. MySQL Setup (Node1)**
        
- Start the MySQL service to check it is working fine.
    
     - From services.msc start the MySQL Service.
     - From services.msc stop the MySQL Service.

### **9. Change the startup type of MySQL Service (Node1 & Node2)**

- Open the Windows Service Manager     
  - On Command Prompt
        > services.msc
- Change the startup type of MySQL Service to Manual.

### **10. Configure MySQL Service in ECX Cluster**
 
  - Add the service resource to control MySQL and configure in Config mode of the Cluster WebUI.  
  - In Config mode of the Cluster WebUI, execute Apply the Configuration File.

    
      
### **11. Testing Of MySQL Database on ECX Cluster**

- MySQL Setup (Node1)
        
    - Click on Start Tab and Open MySQL 8.0 Client Line.
      - Enter password:- P@ssw0rd (Administrative Credential)

    - Verify the location of database 
         
```
    mysql> select @@datadir;
    +----------------+
    | @@datadir      |
    +----------------+
    | D:\MySQL\Data\ |
    +----------------+
    1 row in set (0.00 sec)
```

- Check the MySQL database


```
    mysql> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | sys                |
    +--------------------+
    4 rows in set (0.14 sec)
```         
 
            
- **Create Database in MySql 8.0 Client Line for testing database on Node1.**

 ```
    mysql> create database test1;
    Query OK, 1 row affected (0.03 sec)

    mysql> use test1;
    Database changed

    mysql> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | sys                |
    | test1              |
    +--------------------+
    5 rows in set (0.14 sec)
```    

- Create table 

``` 
    mysql> create table demo160(id int, name varchar(10));
    Query OK, 0 rows affected (0.13 sec)

    mysql> create index id_index on demo160(id);
           Query OK, 0 rows affected (0.17 sec)
           Records: 0  Duplicates: 0  Warnings: 0
```

- Insert the values in the tables

```
    mysql> insert into demo160 values(1, 'harsh');
            Query OK, 1 row affected (0.06 sec)
    mysql> insert into demo160 values(2, 'mukesh');
            Query OK, 1 row affected (0.01 sec)

```
- Verify the created table in MySQL database

```
    mysql> select * from demo160;
           +------+--------+
           | id   | name   |
           +------+--------+
           |    1 | harsh  |
           |    2 | mukesh |
           +------+--------+
           3 rows in set (0.00 sec)
```




### **12. You have to move failover group on the Node2 for testing Mysql database failover.**

         
- Check and Confirm the database on MySQL which we have created on node1.

```
    mysql> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | sys                |
    | test1              |
    +--------------------+
    5 rows in set (0.14 sec)
```

   - The data is replicated from primary to secondary server successfully.   
                 
### **13. Verification**

- Confirm that we can access the database where it failover group is running.
