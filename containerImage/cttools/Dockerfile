FROM rockylinux/rockylinux
MAINTAINER weiwendi WechartPublic: sretech

USER root

COPY kubectl /usr/local/bin/
COPY helm /usr/local/bin/
COPY ct /usr/local/bin/

RUN mkdir /etc/ct/
COPY etc/chart_schema.yaml /etc/ct/
COPY etc/lintconf.yaml /etc/ct/

RUN chmod a+x /usr/local/bin/* && dnf -y install epel-release git-all python38 && dnf -y install yamllint && dnf clean all

COPY Yamale-master /opt/Yamale-master
RUN cd /opt/Yamale-master && python3 setup.py install

USER jenkins
