# create-docker-image-mysql
In this example, I am showing how to generate a Docker image of a MySql database with all the changes implemented using Liquibase changelogs. This will be accomplished using Liquibase Maven plugin to generate the SQL scripts, and fabric8 Maven plugin to generate the Docker image.

In this example, we are reusing the changelogs we created in the [create MySql database with liquibase example](https://github.com/rafatoba/liquibase-examples/tree/create-submodules/bare-liquibase-mysql).

So with all the changelogs in place, we want to do the next:
* Generate SQL scripts out of the changelogs via Liquibase Maven Plugin
* Create an image from a MySql database containing those changes.

### Generate SQL scripts out of the changelogs via Liquibase Maven Plugin ###
Compared with the previous example, in which we told liquibase to connect to a runing MySql database and perform changes there, this time we want to do the things in a different way:
* Generate SQL scripts instead of performing changes in a DB.
* Not perform the changes against any DB (just generate scripts).

#### Generate SQL scripts instead of performing changes in a DB. ####
In order to tell the liquibase Maven plugin that we wanted to change things in a DB, we specified that we wanted to perform the goal ```update``` in the plugin declaration in the pom.xml.
```
			<plugin>
				<groupId>org.liquibase</groupId>
				<artifactId>liquibase-maven-plugin</artifactId>
				<version>3.0.5</version> 
        ...
				<executions>
					<execution>
						<phase>process-resources</phase>
						<goals>
							<goal>update</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
```
What we are going to do is to change the goal to be ```updateSQL```
```
			<plugin>
				<groupId>org.liquibase</groupId>
				<artifactId>liquibase-maven-plugin</artifactId>
				<version>3.6.1</version>
        ...
				<executions>
					<execution>
						<phase>process-resources</phase>
						<goals>
							<goal>updateSQL</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
```
This will tell the plugin to generate the SQL scripts in the /target/liquibase directory instead of trying to perform changes in a DB. 
Also not that I have pumped up the plugin version from 3.0.5 to 3.6.1. This is because for the next step, I am going to be using a feature that is available since version 3.1 of the liquibase Maven plugin: ```offline mode```
### Not perform the changes against any DB (just generate scripts). ###
In the previous step, we have told liquibase to generate the SQL scripts out of the changelog files. However, it still expects to have a connection to a Database, as it was described in liquibase.properties. That is because it wants to check for the existence of the table DATABASECHANGELOG in the schema we want to access for if we have already ran some liquibase changelogs before. In this way, it will only generate the SQL scripts after the last changes that were in that table.
However, what we want is to generate the SQL scripts for *all* the changelogs, because we will generate the database from scratch. So actually we don't want to need any DB access when we run this goal. In order to do so, we need to use the ```offline mode```. For that, you need to change the URL in the file liquibase.properties(in our case, we are calling it liquibase-docker.properties):
```
url:offline:mysql?outputLiquibaseSql=all

```
With this, it won't try to check anything on any MySql database, but it will limit itself to create the scripts. ```outputLiquibaseSql=all``` will force the generation of the DATABASECHANGELOG also. In this way, we will be able to apply new changelogs via liquibase in the future if we want to.

There is one thing a bit bothersome: it will always generate a file called '''databasechangelog.csv''' in the folder we called maven. This file records the changelogs that have been run. So if we don't erase it, it won't generate the SQL scripts for the changelogs that are already there the next time we call ```mvn clean install```.
In order to avoid that problematic, I have declared the clean plugin in the pom.xml, so it now not only deletes the target directory but also that file if present:
```
			<plugin>
				<artifactId>maven-clean-plugin</artifactId>
				<version>3.1.0</version>
				<configuration>
					<filesets>
						<fileset>
							<directory>.</directory>
							<includes>
								<include>**/databasechangelog.csv</include>
							</includes>
							<followSymlinks>false</followSymlinks>
						</fileset>
					</filesets>
				</configuration>
			</plugin>
```
So if we run clean, it will always generate the SQL using all the changelogs we have created.

### Create an image from a MySql database containing those changes. ###
Now we want to generate an image that contains a MySql database already initialized with the MySql changes that liquibase generated. That is possible thanks to the mysql image. The containers created from this image will take the scripts that are in the directory `/docker-entrypoint-initdb.d` and initialize the DB with them. So the Dockerfile will look like the following one:
```
FROM mysql
ENV MYSQL_DATABASE company
ENV MYSQL_ROOT_PASSWORD pass
COPY ./target/liquibase/ /docker-entrypoint-initdb.d/
```
Here, we are telling that we want our image to use ```mysql``` image as a base. We are also going to set the DB name to `company`. We are also setting the super safe password `pass` for the root user. And we are copying the scripts we generated with liquibase (which are now in the directory `target/liquibase/`) into the directory `/docker-entrypoint-initdb.d/` inside the container. In that way, during the container initialization, those scripts will be used to generate the DB data for us.
