# rice-example-config
Some example configuration for various Kuali Rice configuration scenarios

## Standalone Rice Server Configuration

To configure a Rice standalone server:

1. Create a config file according to the [rice-config-standalone.xml](rice-config-standalone.xml) template. Be sure to read the comments in the file as they have important information on how to configure each value which needs to be changed.
2. Name it `common-config.xml` and place in a directory of your choosing.
3. Modify the startup of the Tomcat server (using CATALINA_OPTS or another mechanism) so that it passes the following option to the JVM
```
-Dexternal.config.home=/directory/of/your/choosing
```
Where `external.config.home` points to the directory where you put `common-config.xml`

## Running the Integration Tests

To run Rice integration tests against MySQL

1. Create a config file according to the [rice-config-integration-tests.xml](rice-config-integration-tests.xml] template.
2. Name it 'integration-test-config.xml`
3. From root of Kuali Rice source code, run the following command to bootstrap a new MySQL database for the integration tests (the MySQL server will need to be running and you will need to know the username and password of a MySQL account that has permissions to create and drop databases). Change the parameters to the command as needed for your particular database:
```
mvn -f db/impex/master clean install \
     -Pdb,mysql,integration-test \
     -Dimpex.dba.url=jdbc:mysql://localhost \
     -Dimpex.dba.username=root \
     -Dimpex.dba.password=myrootpassword \
     -Dimpex.username=riceit \
     -Dimpex.password=riceit \
     -Dimpex.database=riceit
``` 
4. Once the database has been created successfully, execute the following to run the integration tests:
```
mvn verify -Pitests -Dbuild.alt.config.location=/path/to/integration-test-config.xml
```
5. Note that the tests will take 1.5 - 4 hours to run depending on the speed of the machine they are being run on as well as the database.
6. Even if there are failed integration tests, the build will still produce a "Build Successful" result. The specific report files will be located in the following directory pattern that can be configured for Junit report files in Jenkins: `**/target/failsafe-reports/*.xml`
7. If after running this on a local machine you would like to generate an HTML report of failures, you can run the following and the resulting report will be in `target/site/failsafe-report.html`:
```mvn surefire-report:failsafe-report-only -Daggregate=true```