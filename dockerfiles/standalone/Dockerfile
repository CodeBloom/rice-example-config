FROM tomcat:8-jre8

ENV APP_CFG_BASE /config
ENV CATALINA_OPTS -Dexternal.config.home=$APP_CFG_BASE
ENV MYSQL_DRIVER_URL https://repo1.maven.org/maven2/mysql/mysql-connector-java/5.1.37/mysql-connector-java-5.1.37.jar
ENV TOMCAT_INSTRUMENT_URL https://repo1.maven.org/maven2/org/springframework/spring-instrument-tomcat/4.2.2.RELEASE/spring-instrument-tomcat-4.2.2.RELEASE.jar

COPY rice-standalone.war $CATALINA_HOME/webapps/ROOT.war

RUN rm -rf $CATALINA_HOME/webapps/ROOT \
    && mkdir $CATALINA_HOME/webapps/ROOT \
    && unzip $CATALINA_HOME/webapps/ROOT.war -d $CATALINA_HOME/webapps/ROOT \
    && rm $CATALINA_HOME/webapps/ROOT.war \
    && cd $CATALINA_HOME/lib \
    && curl -fSL "$MYSQL_DRIVER_URL" -O \
    && curl -fSL "$TOMCAT_INSTRUMENT_URL" -O \
    && cd $CATALINA_HOME

    
VOLUME [$APP_CFG_BASE]
