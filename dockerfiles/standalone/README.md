To build:

First make sure there is a copy of the rice war in the directory with the name
"rice-standalone.war", there is a helper script named "copy-war.sh" which you
can look at. *Note:* Until the customized copy of the session-managers
dependency can be uploaded to a Maven repository you will need to
[check it out](https://github.com/CodeBloom/session-managers) and install it
locally before you can fully prepare this Docker image.  To build the image:

```docker build -t rice-standalone .```

To run:

```docker run -it --rm -v ~/kuali-appconfig:/config --add-host="localhost:10.0.2.2" -p 8080 rice-standalone```

Where ~/kuali-appconfig in this case is where your common-config.xml,
log4j.properties, and rice.jks files are located

If you would like to use Redis-backed sessions which will survive the Docker container restarting, you should create a context.xml file similar to the one [found in the root](../../context.xml) which is configured to point to your Redis instance.  From there you should mount the file in the Docker container at `/usr/local/tomcat/conf/context.xml` like so:

```docker run -it --rm -v ~/kuali-appconfig:/config -v ~/kuali-appconfig/context.xml:/usr/local/tomcat/conf/context.xml --add-host="localhost:10.0.2.2" -p 8080 rice-standalone```
