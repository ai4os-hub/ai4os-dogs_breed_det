# Dockerfile has three Arguments: tag, pyVer, branch
# tag - tag for Tensorflow Image (default: 1.10.0-py3)
# pyVer - python versions as 'python' or 'python3' (default: python3)
# branch - user repository branch to clone (default: master, other option: test)
#
# To build the image:
# $ docker build -t <dockerhub_user>/<dockerhub_repo> --build-arg arg=value docker/
# or using default args:
# $ docker build -t <dockerhub_user>/<dockerhub_repo>  docker/


# DEEPaaS API V2 requires python3.6
# set this tag for our custom built TF image
ARG tag=1.12.0-py36

# Base image, e.g. tensorflow/tensorflow:1.10.0
# DEEPaaS API V2 requires python3.6,
# use our custom built Tensorflow images
FROM deephdc/tensorflow:${tag}

LABEL maintainer='V.Kozlov (KIT)'
# Dogs breed detector based on deep learning. Uses DEEPaaS API

# default is python3 now
ARG pyVer=python3

# What user branch to clone (!)
ARG branch=main

# Oneclient version, has to match OneData Provider and Linux version
ARG oneclient_ver=19.02.0.rc2-1~bionic

# Install ubuntu updates and python related stuff
# link python3 to python, pip3 to pip, if needed
RUN DEBIAN_FRONTEND=noninteractive apt-get update && \
    apt-get install -y --no-install-recommends \
         git \
         graphviz \
         curl \
         wget \
         $pyVer-setuptools \
         $pyVer-pip \
         $pyVer-dev \
         $pyVer-wheel && \ 
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf /root/.cache/pip/* && \
    rm -rf /tmp/* && \
    if [ "$pyVer" = "python3" ] ; then \
       ln -s /usr/bin/pip3 /usr/bin/pip && \
       if [ ! -e /usr/bin/python ]; then \
          ln -s /usr/bin/python3 /usr/bin/python; \
       fi; \
    fi && \
    python --version && \
    pip3 install --no-cache-dir --upgrade pip "setuptools<60.0.0" wheel && \
    pip --version

# Set LANG environment
ENV LANG C.UTF-8

# Set the working directory
WORKDIR /srv

# install rclone
RUN wget https://downloads.rclone.org/rclone-current-linux-amd64.deb && \
    dpkg -i rclone-current-linux-amd64.deb && \
    apt install -f && \
    mkdir /srv/.rclone/ && touch /srv/.rclone/rclone.conf && \
    rm rclone-current-linux-amd64.deb && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf /tmp/*

ENV RCLONE_CONFIG=/srv/.rclone/rclone.conf

# INSTALL oneclient for ONEDATA
RUN curl -sS  http://get.onedata.org/oneclient-1902.sh  | bash -s -- oneclient="$oneclient_ver" && \
    apt-get clean && \
    mkdir -p /mnt/onedata && \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf /tmp/* 

# Disable FLAAT authentication by default
ENV DISABLE_AUTHENTICATION_AND_ASSUME_AUTHENTICATED_USER yes

# EXPERIMENTAL: install deep-start script
# N.B.: This repository also contains run_jupyter.sh
RUN git clone https://github.com/deephdc/deep-start /srv/.deep-start && \
    ln -s /srv/.deep-start/deep-start.sh /usr/local/bin/deep-start

# Necessary for the Jupyter Lab terminal
ENV SHELL /bin/bash

# Install user app:
RUN git clone -b $branch --depth 1 https://github.com/ai4os-hub/ai4os-dogs_breed_det && \
    cd  ai4os-dogs_breed_det && \
    pip install --no-cache-dir -e . && \
    rm -rf /root/.cache/pip/* && \
    rm -rf /tmp/* && \
    cd ..

# Open DEEPaaS port
EXPOSE 5000

# Open Monitoring  and Jupyter ports
EXPOSE 6006 8888

CMD ["deepaas-run", "--listen-ip", "0.0.0.0", "--listen-port", "5000"]
