FROM ubuntu:14.04 
MAINTAINER tobilg <tobilg@gmail.com>

ARG buildno
ARG buildurl

# Add R list
RUN echo 'deb http://cran.rstudio.com/bin/linux/ubuntu trusty/' | sudo tee -a /etc/apt/sources.list.d/r.list && \
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E084DAB9

RUN apt-get update && apt-get install -yq --no-install-recommends --force-yes software-properties-common && add-apt-repository ppa:openjdk-r/ppa && add-apt-repository ppa:fkrull/deadsnakes

# packages
RUN apt-get update && apt-get install -yq --no-install-recommends --force-yes \
    wget \
    git \
    openjdk-8-jdk \
    maven \
    libjansi-java \
    libsvn1 \
    libcurl3 \
    libsasl2-modules && \
    rm -rf /var/lib/apt/lists/*

# Overall ENV vars
ENV SPARK_VERSION 2.3.0
ENV MESOS_BUILD_VERSION 0.28.0-2.0.16
ENV LIVY_BUILD_VERSION $buildno
ENV LIVY_BUILD_URL $buildurl

# Set install path for Livy
ENV LIVY_APP_PATH /apps/livy

# Set build path for Livy
ENV LIVY_BUILD_PATH /apps/build/livy

# Set Hadoop config directory
ENV HADOOP_CONF_DIR /etc/hadoop/conf

# Set Spark home directory
ENV SPARK_HOME /usr/local/spark

# Set native Mesos library path
ENV MESOS_NATIVE_JAVA_LIBRARY /usr/local/lib/libmesos.so

# Python install
RUN apt-get update && apt-get install -y \
    python3.5 \
    python3-setuptools \
    python3-pip

RUN ln -s pip3 /usr/bin/pip
RUN ln -s python3.5 /usr/bin/python

RUN echo $(python --version)
RUN python -m pip install --upgrade pip setuptools wheel

# Mesos install
RUN wget http://repos.mesosphere.com/ubuntu/pool/main/m/mesos/mesos_$MESOS_BUILD_VERSION.ubuntu1404_amd64.deb && \
    dpkg -i mesos_$MESOS_BUILD_VERSION.ubuntu1404_amd64.deb && \
    rm mesos_$MESOS_BUILD_VERSION.ubuntu1404_amd64.deb

# Spark ENV vars
ENV SPARK_VERSION_STRING spark-$SPARK_VERSION-bin-hadoop2.6
ENV SPARK_DOWNLOAD_URL https://www.apache.org/dist/spark/spark-2.3.0/$SPARK_VERSION_STRING.tgz

# Download and unzip Spark
RUN wget $SPARK_DOWNLOAD_URL && \
    mkdir -p $SPARK_HOME && \
    tar xvf $SPARK_VERSION_STRING.tgz -C /tmp && \
    cp -rf /tmp/$SPARK_VERSION_STRING/* $SPARK_HOME && \
    rm -rf -- /tmp/$SPARK_VERSION_STRING && \
    rm spark-$SPARK_VERSION-bin-hadoop2.6.tgz

# 2018-06-16 Download Lnd unzip livy binary
RUN mkdir -p $LIVY_BUILD_PATH/assembly/target && \
    cd $LIVY_BUILD_PATH/assembly/target && \
    wget -O $LIVY_BUILD_VERSION.zip $LIVY_BUILD_URL && \
    unzip $LIVY_BUILD_PATH/assembly/target/$LIVY_BUILD_VERSION.zip -d /apps && \
    mv /apps/livy-0.5.0-incubating-bin $LIVY_APP_PATH && \
    rm -rf $LIVY_BUILD_PATH && \
	mkdir -p $LIVY_APP_PATH/upload && \
	mkdir -p $LIVY_APP_PATH/logs

# Set Alternative java version
RUN update-java-alternatives --set /usr/lib/jvm/java-1.8.0-openjdk-amd64

# Add custom files, set permissions
ADD entrypoint.sh .

RUN chmod +x entrypoint.sh

# Expose port
EXPOSE 8998

ENTRYPOINT ["/entrypoint.sh"]
