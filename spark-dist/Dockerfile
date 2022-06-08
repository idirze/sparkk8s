ARG java_image_tag=11-jre-slim

FROM openjdk:${java_image_tag}

LABEL maintainer="Idir IZITOUNENE <idirze@gmail.com>"

ARG spark_user=spark
ENV SPARK_VERSION=3.0.2
ENV HADOOP_VERSION=3.2
ENV SPARK_HOME=/opt/spark
# Fix the value of PYTHONHASHSEED
# Note: this is needed when you use Python 3.3 or greater
ENV PYTHONHASHSEED 1

RUN set -ex \
    && sed -i 's/http:\/\/deb.\(.*\)/https:\/\/deb.\1/g' /etc/apt/sources.list \
    && apt-get update \
    && ln -s /lib /lib64 \
    && apt install -y bash tini libc6 libpam-modules krb5-user libnss3 procps wget \
    && rm /bin/sh \
    && ln -sv /bin/bash /bin/sh \
    && echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su \
    && chgrp root /etc/passwd && chmod ug+rw /etc/passwd \
    && rm -rf /var/cache/apt/*
    
RUN   wget https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz \
      && tar -xvzf spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz \
      && mv spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION} ${SPARK_HOME} \
      && cp ${SPARK_HOME}/kubernetes/dockerfiles/spark/entrypoint.sh ${SPARK_HOME} \
      && mkdir ${SPARK_HOME}/work-dir \
      && chmod g+w ${SPARK_HOME}/work-dir \
      && cp ${SPARK_HOME}/conf/log4j.properties.template ${SPARK_HOME}/conf/log4j.properties \
      && rm spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz

RUN groupadd -r ${spark_user} --gid=183 \
    && useradd -r -g ${spark_user} --uid=183 -d ${SPARK_HOME} ${spark_user} \
    && chown ${spark_user}:${spark_user} -R ${SPARK_HOME} \
    && chown ${spark_user}:${spark_user} ${SPARK_HOME}/entrypoint.sh \
    && chmod +x ${SPARK_HOME}/entrypoint.sh

USER ${spark_user}

WORKDIR ${SPARK_HOME}/work-dir

ENTRYPOINT [ "/opt/spark/entrypoint.sh" ]

