This container can be used to create a database for Kuali Rice integration
tests against MySQL. You must have MySQL installed and running on localhost.

To build it:

```docker build -t rice:database-mysql-itests .```

To run it:

```
docker run -it --rm -v ~/git/rice:/rice \
   --add-host="localhost:10.0.2.2" \
   -e "DBA_USERNAME=root" \
   -e "DBA_PASSWORD=NONE" \
   -e "DB_USERNAME=RICECI" \
   -e "DB_PASSOWRD=RICECI" \
   -e "DB_NAME=RICECI" \
   rice:database-mysql-itests
```

In the example above, we are using docker-machine so the --add-host allows for
mapping the execution of the container against a mysql server running on
localhost. Modify the environment variables above to match your mysql database
administrative username/password as well as username, password, and database
name you want to use for the generated integration test database. Also, the 
`/rice` volume should point to a copy of the Rice source code containing the
version of the database you want to load.