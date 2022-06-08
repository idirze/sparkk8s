ARG java_image_tag=11-jdk-slim

FROM openjdk:${java_image_tag}

ENV DEBIAN_FRONTEND=noninteractive
ENV MVN_VERSION=3.8.5

RUN apt-get -qq update && \
    apt-get -qq upgrade -y && \
    apt-get -qq install -y git wget curl && \
    wget -q https://dlcdn.apache.org/maven/maven-3/$MVN_VERSION/binaries/apache-maven-$MVN_VERSION-bin.tar.gz -P /tmp && \
    tar xf /tmp/apache-maven-$MVN_VERSION-bin.tar.gz -C /opt && \
    ln -s /opt/apache-maven-$MVN_VERSION /opt/maven && \
    apt-get -qq install gnupg && \
    echo "deb https://repo.scala-sbt.org/scalasbt/debian /" | tee -a /etc/apt/sources.list.d/sbt.list && \
    curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | apt-key add && \
    apt-get -qq update && \
    apt-get -qq install sbt && \
    apt-get -qq install -y r-base && \
    apt install -y python3 python3-pip && \
    pip install --upgrade pip setuptools

ENV M2_HOME=/opt/maven
ENV MAVEN_HOME=/opt/maven
ENV PATH=${PATH}:${M2_HOME}/bin
