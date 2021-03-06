FROM ubuntu:latest

ENV DEBIAN_FRONTEND=noninteractive \
    TZ=Asia/Kolkata

# Java 8
RUN sed -i -e 's/http:\/\/archive/mirror:\/\/mirrors/' -e 's/http:\/\/security/mirror:\/\/mirrors/' -e 's/\/ubuntu\//\/mirrors.txt/' /etc/apt/sources.list && \
    echo "Acquire {http {Timeout \"60\";}; ftp {Timeout \"60\";};};" > /etc/apt/apt.conf.d/custom-apt.conf && \
    apt-get update && apt-get dist-upgrade --yes && apt-get install -y openjdk-8-jdk git tzdata wget python && \
    ln -fs /usr/share/zoneinfo/$TZ /etc/localtime && dpkg-reconfigure -f noninteractive tzdata && \
    apt-get autoremove -y && apt-get clean && rm -rf /var/cache/oracle-jdk8-installer && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /

# Set version and github repo which you want to build from
ARG DRUID_VERSION=0.15.1-incubating

RUN wget https://www-eu.apache.org/dist/incubator/druid/${DRUID_VERSION}/apache-druid-${DRUID_VERSION}-bin.tar.gz && \
      tar -xzf apache-druid-${DRUID_VERSION}-bin.tar.gz && \
      mv apache-druid-${DRUID_VERSION} druid

WORKDIR /druid

# Expose ports:
# - 8888: HTTP (router)
# - 8081: HTTP (coordinator)
# - 8082: HTTP (broker)
# - 8083: HTTP (historical)
# - 8090: HTTP (overlord)
# - 8091: HTTP (middlemanager)
EXPOSE 8081
EXPOSE 8888
EXPOSE 8082
EXPOSE 8083
EXPOSE 8090
EXPOSE 8091

COPY conf /druid/conf

ENTRYPOINT [ "/bin/bash" ]
