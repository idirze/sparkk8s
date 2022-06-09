ARG java_image_tag=11-jre-slim

FROM openjdk:${java_image_tag}

LABEL maintainer="Idir IZITOUNENE <idirze@gmail.com>"

ARG spark_uid=185
ENV SPARK_VERSION=3.1.1
ENV HADOOP_VERSION=3.2
ENV SPARK_HOME=/opt/spark
ENV SPARK_CONF_DIR  $SPARK_HOME/conf
ENV SPARK_WORK_DIR  $SPARK_HOME/work-dir
# Fix the value of PYTHONHASHSEED
# Note: this is needed when you use Python 3.3 or greater
ENV PYTHONHASHSEED 1

RUN set -ex \
    && sed -i 's/http:\/\/deb.\(.*\)/https:\/\/deb.\1/g' /etc/apt/sources.list \
    && apt-get update \
    && ln -s /lib /lib64 \
    && apt install -y bash tini libc6 libpam-modules krb5-user libnss3 procps wget --no-install-recommends \
    && rm /bin/sh \
    && ln -sv /bin/bash /bin/sh \
    && echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su \
    && chgrp root /etc/passwd && chmod ug+rw /etc/passwd \
    && rm -rf /var/cache/apt/* \
    && rm -rf /var/lib/apt/lists/*
    
RUN   wget https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz \
      && tar -xvzf spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz \
      && mv spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION} ${SPARK_HOME} \
      && cp ${SPARK_HOME}/kubernetes/dockerfiles/spark/entrypoint.sh ${SPARK_HOME} \
      && mkdir ${SPARK_HOME}/work-dir \
      && chmod g+w ${SPARK_HOME}/work-dir \
      && cp ${SPARK_HOME}/conf/log4j.properties.template ${SPARK_HOME}/conf/log4j.properties \
      && rm spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz

WORKDIR ${SPARK_HOME}/work-dir
RUN chmod g+w $SPARK_WORK_DIR

ENTRYPOINT [ "/opt/spark/entrypoint.sh" ]

USER ${spark_uid}