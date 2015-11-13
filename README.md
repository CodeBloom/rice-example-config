# rice-example-config
Some example configuration for various Kuali Rice configuration scenarios

## Standalone Rice Server Configuration

To configure a Rice standalone server:

# Create a config file according to the [rice-config-standalone.xml](rice-config-standalone.xml) template. Be sure to read the comments in the file as they have important information on how to configure each value which needs to be changed.
# Name it `common-config.xml` and place in a directory of your choosing.
# Modify the startup of the Tomcat server (using CATALINA_OPTS or another mechanism) so that it passes the following option to the JVM ```-Dexternal.config.home=/directory/of/your/choosing``` Where `external.config.home` points to the directory where you put `common-config.xml`
