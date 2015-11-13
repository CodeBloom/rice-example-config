To build:

First make sure there is a copy of the rice war in the directory with the name
"rice-standalone.war", there is a helper script named "copy-war.sh" which you
can look at.

```docker build -t rice-standalone .```

To run:

```docker run -it --rm -v ~/kuali-appconfig:/config --add-host="localhost:10.0.2.2" -p 8080 rice-standalone```

Where ~/kuali-appconfig in this case is where your common-config.xml,
log4j.properties, and rice.jks files are located
