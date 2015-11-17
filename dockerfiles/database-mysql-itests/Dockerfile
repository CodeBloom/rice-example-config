FROM maven:3-jdk-8

ENV DBA_USERNAME root
ENV DBA_PASSWORD NONE
ENV DB_USERNAME RICECI
ENV DB_PASSWORD RICECI
ENV DB_NAME RICECI

VOLUME /rice

CMD mvn -f /rice/db/impex/master clean install \
     -Pdb,mysql,integration-test \
     -Dimpex.dba.username=$DBA_USERNAME \
     -Dimpex.dba.password=$DBA_PASSWORD \
     -Dimpex.username=$DB_USERNAME \
     -Dimpex.password=$DB_PASSWORD \
     -Dimpex.database=$DB_NAME
