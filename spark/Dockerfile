ARG java_image_tag=11-jre-slim

### Builder Container
FROM idirze/spark-builder:1.0.0 as builder

ARG SPARK_VERSION=3.1.1
ARG HADOOP_VERSION=3.2
ARG SCALA_VERSION=2.12

ENV MAVEN_OPTS "-Xss64m -Xmx2g -XX:ReservedCodeCacheSize=1g"

RUN cd / && \
    git clone https://github.com/apache/spark.git --branch v${SPARK_VERSION} --single-branch && \
    cd /spark && \
    dev/make-distribution.sh \
        --name hadoop-${HADOOP_VERSION}-cloud-scala-${SCALA_VERSION} --tgz \
        -Phadoop-${HADOOP_VERSION} \
        -Pscala-${SCALA_VERSION} \
        -Pkubernetes \
        -Phive \
        -Phive-thriftserver && \
    cp spark-${SPARK_VERSION}-bin-hadoop-${HADOOP_VERSION}-cloud-scala-${SCALA_VERSION}.tgz /

RUN mkdir /spark-dist && \
    cd /spark-dist && \
    mv /spark-${SPARK_VERSION}-bin-hadoop-${HADOOP_VERSION}-cloud-scala-${SCALA_VERSION}.tgz  /spark-dist && \
    tar -zxvf spark-${SPARK_VERSION}-bin-hadoop-${HADOOP_VERSION}-cloud-scala-${SCALA_VERSION}.tgz && \
    mv spark-${SPARK_VERSION}-bin-hadoop-${HADOOP_VERSION}-cloud-scala-${SCALA_VERSION} spark
    
### Final Container
FROM openjdk:${java_image_tag}

LABEL maintainer="Idir Izitounene <idirze@gmail.com>"

ARG SPARK_VERSION=3.1.1
ARG HADOOP_VERSION=3.2
ARG SCALA_VERSION=2.12
ARG spark_uid=185

RUN set -ex && \
    sed -i 's/http:\/\/deb.\(.*\)/https:\/\/deb.\1/g' /etc/apt/sources.list && \
    apt-get update && \
    ln -s /lib /lib64 && \
    apt install -y bash tini libc6 libpam-modules krb5-user libnss3 procps --no-install-recommends && \
    rm /bin/sh && \
    ln -sv /bin/bash /bin/sh && \
    echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su && \
    chgrp root /etc/passwd && chmod ug+rw /etc/passwd && \
    rm -rf /var/cache/apt/* && \
    rm -rf /var/lib/apt/lists/*

ENV SPARK_VERSION   ${SPARK_VERSION}
ENV HADOOP_VERSION  ${HADOOP_VERSION}
ENV SCALA_VERSION   ${SCALA_VERSION}

ENV SPARK_HOME      /opt/spark
ENV SPARK_CONF_DIR  $SPARK_HOME/conf
ENV SPARK_WORK_DIR  $SPARK_HOME/work-dir

### install spark
COPY --from=builder /spark-dist/spark $SPARK_HOME
RUN mkdir -p $SPARK_HOME/work-dir && \
    mkdir -p $SPARK_HOME/spark-warehouse

COPY entrypoint.sh /opt/

WORKDIR $SPARK_WORK_DIR
RUN chmod g+w $SPARK_WORK_DIR

ENTRYPOINT [ "/opt/entrypoint.sh" ]

# Specify the User that the actual main process will run as
USER ${spark_uid}
