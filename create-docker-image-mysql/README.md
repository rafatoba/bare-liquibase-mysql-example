# create-docker-image-mysql
In this example, I am showing how to generate a Docker image of a MySql database with all the changes implemented using Liquibase changelogs. This will be accomplished using Liquibase Maven plugin to generate the SQL scripts, and fabric8 Maven plugin to generate the Docker image.

In this example, we are reusing the changelogs we created in the [create MySql database with liquibase example](https://github.com/rafatoba/liquibase-examples/tree/create-submodules/bare-liquibase-mysql).

So with all the changelogs in place, we want to do the next:
* Generate SQL scripts out of the changelogs via Liquibase Maven Plugin
* Create an image from a MySql database containing those changes.
