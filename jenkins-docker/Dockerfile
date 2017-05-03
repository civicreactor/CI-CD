FROM jenkins:2.46.2-alpine

EXPOSE 8080 5000

VOLUME /var/jenkins_home 
WORKDIR /var/jenkins_home 


# Logging configuration
ENV LOGGING_CONF handlers=java.util.logging.ConsoleHandler \
jenkins.level=FINEST \
java.util.logging.ConsoleHandler.level=FINEST \

# Java configuration
#ENV JAVA_OPTS

RUN mkdir data && echo $LOGGING_CONF > data/log.properties

# Security configuration
#COPY https.pem /var/lib/jenkins/cert
#COPY https.key /var/lib/jenkins/pk
#ENV JENKINS_OPTS --httpPort=-1 --httpsPort=8083 --httpsCertificate=/var/lib/jenkins/cert --httpsPrivateKey=/var/lib/jenkins/pk
#EXPOSE 8083

COPY plugins.txt /usr/share/jenkins/plugins.txt
RUN /usr/local/bin/plugins.sh /usr/share/jenkins/plugins.txt
