FROM openjdk:8

ENV SONAR_VERSION=6.7 \
    SONARQUBE_HOME=/opt/sonarqube \
    # Database configuration
    # Defaults to using H2
    SONARQUBE_JDBC_USERNAME=sonar \
    SONARQUBE_JDBC_PASSWORD=sonar \
    SONARQUBE_JDBC_URL=

# Http port
EXPOSE 9000

RUN groupadd -r sonarqube && useradd -r -g sonarqube sonarqube

# grab gosu for easy step-down from root
RUN set -x \
    && wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/1.10/gosu-$(dpkg --print-architecture)" \
    && wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/1.10/gosu-$(dpkg --print-architecture).asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
    && gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
    && rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true

RUN set -x \

    # pub   2048R/D26468DE 2015-05-25
    #       Key fingerprint = F118 2E81 C792 9289 21DB  CAB4 CFCA 4A29 D264 68DE
    # uid                  sonarsource_deployer (Sonarsource Deployer) <infra@sonarsource.com>
    # sub   2048R/06855C1D 2015-05-25
    && gpg --keyserver ha.pool.sks-keyservers.net --recv-keys F1182E81C792928921DBCAB4CFCA4A29D26468DE \

    && cd /opt \
    && curl -o sonarqube.zip -fSL https://sonarsource.bintray.com/Distribution/sonarqube/sonarqube-$SONAR_VERSION.zip \
    && curl -o sonarqube.zip.asc -fSL https://sonarsource.bintray.com/Distribution/sonarqube/sonarqube-$SONAR_VERSION.zip.asc \
    && gpg --batch --verify sonarqube.zip.asc sonarqube.zip \
    && unzip sonarqube.zip \
    && mv sonarqube-$SONAR_VERSION sonarqube \
    && chown -R sonarqube:sonarqube sonarqube \
    && rm sonarqube.zip* \
    && rm -rf $SONARQUBE_HOME/bin/*

# ------------------
# Manual Plugin Install - START
# ------------------
#
# Ref: https://github.com/mpas/docker-sonarqube/blob/master/Dockerfile
#   docker build -t odt-sonar .
#   docker run -d --name odt-sonar -p 9000:9000 -p 9092:9092 odt-sonar
#   docker ps -a
#   docker exec -it <container id> /bin/bash
#
# Manual Installation
# To manually install a commercial edition, see SonarSource Editions.
# In the page dedicated to the plugin you want to install (ex: for Python : SonarPython), click on the "Download" link of the version compatible with your SonarQube version.
# Put the downloaded jar in $SONARQUBE_HOME/extensions/plugins, removing any previous versions of the same plugins.
# Once done, you will need to restart your SonarQube Server.
#
# download plugins from:
# - http://downloads.sonarsource.com/plugins/org/codehaus/sonar-plugins/
# - https://sonarsource.bintray.com/Distribution/
# - https://github.com/SonarQubeCommunity/
#
# Plugins can be downloaded from :
# - https://redirect.sonarsource.com/doc/plugin-library.html
#
RUN set -x \
  && cd $SONARQUBE_HOME/extensions/plugins \
  && wget https://sonarsource.bintray.com/Distribution/sonar-java-plugin/sonar-java-plugin-4.15.0.12310.jar \
  && wget https://github.com/pmayweg/sonar-groovy/releases/download/1.6-RC1/sonar-groovy-plugin-1.6-RC1.jar \
  && wget https://sonarsource.bintray.com/Distribution/sonar-javascript-plugin/sonar-javascript-plugin-3.3.0.5702.jar \
  && wget http://downloads.sonarsource.com/plugins/org/codehaus/sonar-plugins/sonar-timeline-plugin/1.5/sonar-timeline-plugin-1.5.jar \
  && wget https://sonarsource.bintray.com/Distribution/sonar-python-plugin/sslr-python-toolkit-1.8.0.1496.jar \
  && wget https://github.com/SonarQubeCommunity/sonar-timeline/releases/download/sonar-timeline-plugin-1.5/sonar-timeline-plugin-1.5.jar \
  && wget https://sonarsource.bintray.com/Distribution/sonar-scm-git-plugin/sonar-scm-git-plugin-1.3.0.869.jar \
  && wget https://sonarsource.bintray.com/Distribution/sonar-findbugs-plugin/sonar-findbugs-plugin-3.3.jar \
  && wget https://sonarsource.bintray.com/Distribution/sonar-pmd-plugin/sonar-pmd-plugin-2.5.jar \
  && wget https://sonarsource.bintray.com/Distribution/sonar-checkstyle-plugin/sonar-checkstyle-plugin-2.4.jar \
  && wget https://sonarsource.bintray.com/Distribution/sonar-generic-coverage/sonar-generic-coverage-plugin-1.2.jar \
  && wget https://sonarsource.bintray.com/Distribution/sonar-xml-plugin/sonar-xml-plugin-1.4.3.1027.jar \
  && wget https://sonarsource.bintray.com/Distribution/sonar-web-plugin/sonar-web-plugin-2.5.0.476.jar

# ------------------
# Manual Plugin Install - END
# ------------------

VOLUME "$SONARQUBE_HOME/data"

WORKDIR $SONARQUBE_HOME
COPY run.sh $SONARQUBE_HOME/bin/
ENTRYPOINT ["./bin/run.sh"]
