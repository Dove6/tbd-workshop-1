FROM python:3.10-slim-bullseye as base
ARG DBT_VERSION
ARG DBT_SPARK_VERSION
ARG SPARK_VERSION
ARG PIP_VERSION=24.*
ARG SETUPTOOLS_VERSION=69.*
ARG WHEEL_VERSION=0.43.*
ARG HADOOP_CONF_DIR=/etc/hadoop/conf

# System setup
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
  git \
  ssh-client \
  software-properties-common \
  make \
  build-essential \
  ca-certificates \
  libpq-dev \
  python-dev \
  libsasl2-dev \
  gcc \
  unixodbc-dev \
  openjdk-11-jdk \
  procps \
  && apt-get clean \
  && rm -rf \
  /var/lib/apt/lists/* \
  /tmp/* \
  /var/tmp/*

# Env vars
ENV PYTHONIOENCODING=utf-8
ENV LANG=C.UTF-8

# Update python and install packages
RUN pip install --upgrade "pip==$PIP_VERSION" "setuptools==$SETUPTOOLS_VERSION" "wheel==$WHEEL_VERSION" --no-cache-dir \
  "dbt-core==$DBT_VERSION" \
  "dbt-spark==$DBT_SPARK_VERSION" \
  "pyspark==$SPARK_VERSION"

RUN mkdir -p $HADOOP_CONF_DIR
COPY conf/* $HADOOP_CONF_DIR
ENV HADOOP_CONF_DIR=$HADOOP_CONF_DIR
ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
WORKDIR /dbt
COPY profiles.yml .