FROM ubuntu:18.04

ENV Z_VERSION="0.10.1"

ENV LOG_TAG="[ZEPPELIN_${Z_VERSION}]:" \
    ZEPPELIN_HOME="/opt/zeppelin" \
    HOME="/opt/zeppelin" \
    LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8 \
    JAVA_HOME=/usr/lib/jvm/java-8-openjdk-arm64 \
    ZEPPELIN_ADDR="0.0.0.0"

RUN echo "$LOG_TAG install basic packages" && \
    apt-get -y update && \
    # Switch back to install JRE instead of JDK when moving to JDK9 or later.
    DEBIAN_FRONTEND=noninteractive apt-get install -y locales language-pack-en openjdk-8-jdk-headless wget unzip && \
    # Cleanup
    rm -rf /var/lib/apt/lists/* && \
    apt-get autoclean && \
    apt-get clean

# Install conda to manage python and R packages
ARG miniconda_version="py37_4.9.2"
# Hashes via https://docs.conda.io/en/latest/miniconda_hashes.html
ARG miniconda_sha256="ccbac800a2d897218dde1df3711d26299a083ca0beb118edf62cf8f3d9516da8"
# Install python and R packages via conda
COPY env_python_3_with_R.yml /env_python_3_with_R.yml

RUN set -ex && \
    wget -nv https://repo.anaconda.com/miniconda/Miniconda3-${miniconda_version}-Linux-aarch64.sh -O miniconda.sh && \
    echo "${miniconda_sha256} miniconda.sh" > anaconda.sha256 && \
    sha256sum --strict -c anaconda.sha256 && \
    bash miniconda.sh -b -p /opt/conda && \
    export PATH=/opt/conda/bin:$PATH && \
    conda config --set always_yes yes --set changeps1 no && \
    conda info -a && \
    conda install mamba -c conda-forge && \
    mamba env update -f /env_python_3_with_R.yml --prune && \
    # Cleanup
    rm -v miniconda.sh anaconda.sha256  && \
    # Cleanup based on https://github.com/ContinuumIO/docker-images/commit/cac3352bf21a26fa0b97925b578fb24a0fe8c383
    find /opt/conda/ -follow -type f -name '*.a' -delete && \
    find /opt/conda/ -follow -type f -name '*.js.map' -delete && \
    mamba clean -ay
    # Allow to modify conda packages. This allows malicious code to be injected into other interpreter sessions, therefore it is disabled by default
    # chmod -R ug+rwX /opt/conda
ENV PATH /opt/conda/envs/python_3_with_R/bin:/opt/conda/bin:$PATH

RUN echo "$LOG_TAG Download Zeppelin binary" && \
    mkdir -p ${ZEPPELIN_HOME} && \
    wget -nv -O /tmp/zeppelin-${Z_VERSION}-bin-all.tgz https://archive.apache.org/dist/zeppelin/zeppelin-${Z_VERSION}/zeppelin-${Z_VERSION}-bin-all.tgz && \
    tar --strip-components=1 -zxvf  /tmp/zeppelin-${Z_VERSION}-bin-all.tgz -C ${ZEPPELIN_HOME} && \
    rm -f /tmp/zeppelin-${Z_VERSION}-bin-all.tgz && \
    chown -R root:root ${ZEPPELIN_HOME} && \
    mkdir -p ${ZEPPELIN_HOME}/logs ${ZEPPELIN_HOME}/run ${ZEPPELIN_HOME}/webapps && \
    # Allow process to edit /etc/passwd, to create a user entry for zeppelin
    chgrp root /etc/passwd && chmod ug+rw /etc/passwd && \
    # Give access to some specific folders
    chmod -R 775 "${ZEPPELIN_HOME}/logs" "${ZEPPELIN_HOME}/run" "${ZEPPELIN_HOME}/notebook" "${ZEPPELIN_HOME}/conf" && \
    # Allow process to create new folders (e.g. webapps)
    chmod 775 ${ZEPPELIN_HOME} && \
    chmod -R 775 /opt/conda

COPY log4j.properties ${ZEPPELIN_HOME}/conf/
COPY log4j_docker.properties ${ZEPPELIN_HOME}/conf/
COPY log4j2.properties ${ZEPPELIN_HOME}/conf/
COPY log4j2_docker.properties ${ZEPPELIN_HOME}/conf/


### SPARK ######################################################################
ENV SPARK_VERSION=3.1.2 \
    HADOOP_VERSION=2.7 \
    SPARK_HOME=/opt/spark

RUN export DEBIAN_FRONTEND=noninteractive && apt-get update && apt-get install -yq krb5-user libpam-krb5 && apt-get clean
RUN apt-get update && apt-get install -y curl unzip wget grep sed vim tzdata && apt-get clean

RUN wget https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz
RUN tar zxvf spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz
RUN mv spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}/ /opt/spark
RUN rm spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz
################################################################################


USER root
EXPOSE 8080
WORKDIR ${ZEPPELIN_HOME}
CMD ["bin/zeppelin.sh"]
