# bare-liquibase-mysql-example
Simple liquibase example with MySQL. This will generate a simple company database with an employee table using MySQL. For more information regarding liquibase, please visit https://www.liquibase.org/
## Configuration
Out of the box, this project is going to access your local instance of MySQL using user _root_ and password _pass_ . If you want to change this, you will need to edit the file _src/main/resources/liquibase/liquibase.properties_ . The file looks like the next:
```
#liquibase.properties
driver: com.mysql.cj.jdbc.Driver
classpath: lib/mysql-connector-java-5.0.5-bin.jar
url:jdbc:mysql://localhost:3306/company?createDatabaseIfNotExist=true
username: root
password: pass
changeLogFile:changelog-master.xml
logLevel: finest
```
As you see in the URL, it already has present there the name of the database(_company_). Since _createDatabaseIfNotExist_ is _true_, it will try to create the database if you haven't created it manually before. So make sure that whatever user you choose has actually enough rights to create tables and, if you haven't created the database before, databases.

## Generating changelogs
In this example, I am using chagelogs in xml format. You can see an example under _src/main/resources/liquibase/changelog/changelog-table-employee.xml_ . If you want to do any change to the database, please create another changelog file under _src/main/resources/liquibase/changelog/_ , and add it in _src/main/resources/changelog-master.xml_ . You can see how we added _changelog-table-employee.xml_ in the master:
```
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog 
...
   http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.0.xsd">
  <include file="src/main/resources/liquibase/changelog/changelog-table-employee.xml"/>
  <include file="src/main/resources/liquibase/changelog/changelog-content-employee.xml"/> 
</databaseChangeLog>
```
## How to run it
Just go to the root of the project, and run ```mvn clean install``` .
## Using docker to avoid messing with MySQL locally
If you don't have a MySQL installed, or if you don't want to mess with your local MySQL instance, you can create a docker container from the mysql image:
``$ docker run -p 3306:3306 --name hb-mysql-example -e MYSQL_ROOT_PASSWORD=pass -d mysql``
This will create and run a container with a MySQL image, with the password _pass_ for the root user (so it would set it up to run this project out of the box).
If you want to get in the container to run some queries:
``$ docker exec -it hb-mysql-example bash``
``mysql -u root -ppass``
Now you are ready to run the project as we explained above.
