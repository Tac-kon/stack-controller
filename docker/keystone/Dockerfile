FROM ubuntu:22.04

RUN apt-get update \
  && apt-get -y install software-properties-common \
  && add-apt-repository cloud-archive:bobcat
  && apt-get update

RUN apt-get -y install keystone python3-openstackclient libapache2-mod-wsgi-py3 python3-oauth2client
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

EXPOSE 5000
EXPOSE 35357