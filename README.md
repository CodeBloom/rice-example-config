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

## Running Kuali Rice Standalone Server (Clustered)

To run multiple clustered Kuali Rice Standalone instances you need to follow these steps:

1. Create a Redis data store
2. Create and populate a MySQL database
3. Create an S3 Bucket
4. Create a keystore
5. Create a log4j configuration file
6. Create a Rice configuration file
7. Configure a Tomcat Server
8. Deploy Kuali Rice Standalone to Tomcat Server
9. Run Tomcat Servers
10. Hook up a Load Balancer

### Create a Redis data store

In order to run Kuali Rice in a cluster but still allow the backend to be stateless we need to configure session
management in Tomcat using a shared data store. Specifically, we use Redis for this. The Tomcat configuration for this
will be covered in a later step, but in preparation for running Kuali Rice in test, a Redis database must be
provisioned.

### Create and populate a MySQL database

(Note that Rice also supports Oracle but we are using MySQL for this example)

Kuali Rice stores most of it's data in a relational database. In order to create a Kuali Rice database, the following
steps must be taken.

1. Checkout a copy of the Rice source code for the version you are installing.
3. Run the following command, replacing the various parameters with the desired username, password, etc. that you would
like to use:
```
mvn -f db/impex/server/demo clean install \
    -Pdb,mysql \
    -Dimpex.dba.url=jdbc:mysql://localhost \
    -Dimpex.dba.username=root \
    -Dimpex.dba.password=myrootpassword \
    -Dimpex.username=myusername \
    -Dimpex.password=mypassword \
    -Dimpex.database=mydatabase
``` 
4. If your root account has no password use `-Dimpex.dba.password=NONE`
5. In the above example we are using the "demo" dataset for the Kuali Rice Standalone Server. If you
would rather work with the bare minimum Kuali Rice database with no demo data, then use the
`-f db/impex/server/bootstrap` option instead.

### Create an S3 Bucket

Kuali Rice will store attachments in Amazon S3. In order to configure this you will need read and write access to a
bucket within Amazon S3. Create this bucket within Amazon and ensure that you have the credentials when we are ready to
configure it later in the process.

Note: By default Kuali Rice will store attachments on the file system, however in order for Rice to run well within a
cloud-based architecture, S3 is the best option for this storage.

### Create a Keystore

Kuali Rice has numerous web service APIs and it's architecture includes frequent communication with other Kuali Rice
client applications using these APIs. The security mechanism used to authorize these communications leverages
public/private key pairs and digital signatures. These credentials are stored within a Java keystore
https://en.wikipedia.org/wiki/Keystore

To create a keystore file, follow these steps (you will need to have the `keytool` binary on your path, which is a
standard component of the JDK installation):

1. Execute the following to create your keystore file and initial key:
```
keytool -genkey -alias rice -keyalg RSA -keystore rice.jks -dname "cn=rice" -keypass <your key pw> -storepass <your store pw>
```
2. Be sure to enter a password of your choosing for your key and your keystore. This will create a file named `rice.jks`
in your current directory.
3. Next self sign the key by executing the following:
```
keytool -selfcert -alias rice -keystore rice.jks -keypass <your key pw> -storepass <your store pw>
```
4. If you don't want to self sign the key, you can have it signed by a certificate authority if you choose.

If you are going to be integrating Rice with another Kuali application (such as KFS) then you will need to use the 
`exportcert` option of keytool and export a certificate to import and trust into KFS's keystore and vice versa. This
establishes the trust relationship between the two applications.

Once you have the keystore created, be sure to keep it in a safe place, we will need it later.

### Create a log4j configuration file

Create a log4j configuration file. There is an example at [/dockerfiles/standalone/log4j.properties](/dockerfiles/standalone/log4j.properties)

### Create a Rice configuration file

Kuali Rice is configured via an XML file, an example file that is annotated with comments can be found at
(dockerfiles/standalone/rice-config.xml).

### Configure a Tomcat Server

Tomcat 8 should be used to run the Kuali Rice standalone server. It needs to be configured as follows:

1. Download the MySQL JDBC driver from http://search.maven.org/remotecontent?filepath=mysql/mysql-connector-java/5.1.38/mysql-connector-java-5.1.38.jar
and copy it into the `lib` directory of Tomcat.
2. Checkout the https://github.com/KualiCo/session-managers project and run a `mvn package`
3. Copy the resulting file in `redis-store/target/redis-store-x.x.x.BUILD-SNAPSHOT.jar` to the Tomcat `lib` directory

### Deploy Kuali Rice Standalone to Tomcat Server

Download the Kuali Rice Standalone WAR into a file named `ROOT.war`. The Kuali Rice Standalone WAR can be retrieved from
a Maven repository under group ID `org.kuali.rice`, artifact ID `rice-standalone`, and at the required version number.

In order to configure Redis for session management the WAR file will need to be unzipped and patched. Patch the
`META-INF` directory within the unzipped WAR to replace the `context.xml` file contained within with [/context.xml](/context.xml). Be
sure to enter the credentials and connection information for Redis into this file.
 
Once this file has been patched, re-zip the WAR file and deploy the patched `ROOT.war` to the `webapps` directory within
Apache Tomcat.

### Run Tomcat Servers

When running the Tomcat servers a few system parameters need to be passed to the JVM (using the Java `-D` option):

* `-Denvironment=prd`
  * if an environment other than production, use a different environment code
  * the "prd" code is special in that it affects certain features like disabling the ability to perform backdoor logins
    and enabling the sending of emails to real users
* `-Dspring.profiles.active=s3`
  * This enables the use of S3 for attachements in KEW and KNS/KRAD
* `-Dadditional.config.locations=/path/to/rice-config.xml`
  * ensure this points to the rice-config.xml file you set up in a previous step
* `-Dinstance.url=https://x.x.x.x:yyyy`
  * instance URL should be different for each individual instance as it will be used by the KSB to route messages to
    individual instances of the Kuali Rice applications within a cluster

### Hook up a Load Balancer

When running more than one instance of the Kuali Rice Standalone Server, a load balancer will need to be set up to
distribute load to the various instances. Because Tomcat has been setup to manage session information inside of Redis,
there is no need to implement session affinity at the load balancer layer.


