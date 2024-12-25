# k8s-guacamole
Tested in December 2024 working simple guacamole deployment in k8s cluster

prerequisites:
1) Working kubernetes cluster
2) MySQL on a separate machine installed + configured to listen on 0.0.0.0 instead of localhost

Prepare the Database

Download DB Extension to your MySQL machine:
    wget https://wiki.orticongroup.ru/files/Distrib/guacamole/guacamole-auth-jdbc-1.5.5.tar.gz 
    tar -xf guacamole-auth-jdbc-1.5.5.tar.gz
    cd guacamole-auth-jdbc-1.5.5.tar.gz/schema

Inside this folder there will be 2 .sql files that set up the DB schema and create admin user, you will need them to prepare the DB in the following steps

connect to database as root and create a user guacamole, database guacamole_db, grant all privileges for this user on a new db
   
    mysql -u root
    CREATE DATABASE guacamole_db;
    CREATE USER 'guacamole'@'%' IDENTIFIED BY 'guacamole'; #here you create a user guacamole that can connect from anywhere (%) and has password guacamole
    GRANT ALL PRIVILEGES ON guacamole_db.* TO 'guacamole'@'%';
    FLUSH PRIVILEGES;
    use guacamole; #here you change the DB to apply the sql query
    source 001-create-schema.sql # INSERT FULL PATH TO THIS SCHEMA FILE
    source 002-create-admin-user.sql # INSERT FULL PATH TO THIS SCHEMA FILE
    show tables; #check that you have new tables created, all queries must result in Query OK
    quit

Database is ready, go to your control plane in k8s cluster.

git clone https://github.com/LuckyNumber17/k8s-guacamole/

cd k8s-guacamole
nano deployment.yml

Change the database connection settings at the end of the file

          - name: GUACD_HOSTNAME
            value: "guacd"
          - name: GUACD_PORT
            value: "4822"
          - name: MYSQL_HOSTNAME
            value: "" #YOUR MYSQL MACHINE IP HERE
          - name: MYSQL_PORT
            value: "3306" #CHANGE PORT IF NEEDED
          - name: MYSQL_DATABASE
            value: "guacamole_db"
          - name: MYSQL_USER
            value: "guacamole"
          - name: MYSQL_PASSWORD
            value: "guacamole"
          - name: MYSQL_DRIVER
            value: "mysql"

Save the file, then create the deployment. The deployment uses NodePort to expose the guacamole client on port 30088, after the pods startup go to http://[your-worker-ip-node]:30088/guacamole, default creds are: guacadmin/guacadmin
