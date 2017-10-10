# docker-centos-sshd

This document is on how to create the Dockfile and image of CentOS + SSHD.

1.Pre-request:

You have a virtual host (let's call it SVR) with CentOS 7 and Docker installed.
You have already installed a CentOS 7 Docker image in you host.
i.e. docker pull centos:latest

Generete private/public keys if not exists in SVR.
ls /root/.ssh/id_rsa.pub 
If not exists then:
ssh-keygen -q -N "" -t rsa -f /root/.ssh/id_rsa

2.Create Dockerfile

Create the directory cd /root/docker/centos-sshd.
cd /root/docker/centos-sshd
cat ~/.ssh/id_rsa.pub > authorized_keys

Create the file run_sshd.sh like the following:
#!/bin/bash
/usr/sbin/sshd -D

Create Dockerfile like the folloiwng:

FROM centos
MAINTAINER youname youremail@qq.com
WORKDIR /root/docker/centos-sshd
RUN yum install passwd openssl openssh-server -y
RUN yum install vim-enhanced -y
RUN yum install net-tools.x86_64 -y
RUN mkdir -p /root/.ssh
ADD authorized_keys /root/.ssh/authorized_keys
RUN ssh-keygen -q -t rsa -b 2048 -f /etc/ssh/ssh_host_rsa_key -N '' 
RUN ssh-keygen -q -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N ''
RUN ssh-keygen -t dsa -f /etc/ssh/ssh_host_ed25519_key  -N '' 
RUN sed -i "s/#UsePrivilegeSeparation.*/UsePrivilegeSeparation no/g" /etc/ssh/sshd_config
RUN sed -ri 's/session    required     pam_loginuid.so/#session    required     pam_loginuid.so/g' /etc/pam.d/sshd
ADD run_sshd.sh /run_sshd.sh
RUN chmod 755 /run_sshd.sh
EXPOSE 22
CMD ["/run_sshd.sh"]

3.Build and Run:

docker build -t sshd:centos7 .
docker run -d -p 10022:22 sshd:centos7
ssh 127.0.0.1 -p 10022

