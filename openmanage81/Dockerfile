# Use CentOS 7 base image from Docker Hub
FROM centos:centos7
MAINTAINER Jose De la Rosa "https://github.com/jose-delarosa"

# Environment variables
ENV PATH $PATH:/opt/dell/srvadmin/bin:/opt/dell/srvadmin/sbin
ENV TOMCATCFG /opt/dell/srvadmin/lib64/openmanage/apache-tomcat/conf/server.xml
ENV USER root
ENV PASS password

# Do overall update and install missing packages needed for OpenManage
RUN yum -y update && \
    yum -y install gcc wget perl passwd which tar libstdc++.so.6 compat-libstdc++-33.i686 glibc.i686 \
        net-snmp

# Set login credentials
RUN echo "$USER:$PASS" | chpasswd

# Add OMSA repo
RUN wget -q -O - http://linux.dell.com/repo/hardware/latest/bootstrap.cgi | bash

# Let's "install all", however we can select specific components instead
RUN yum -y install srvadmin-all && yum clean all

# Fix storage recognition
RUN sed -i -e 's/\x00release_date\x00/\x00version\x00\x00\x00\x00\x00\x00/' /opt/dell/srvadmin/lib64/libstorelib.so.4.20.1-0

# Replace weak Diffie-Hellman ciphers with Elliptic-Curve Diffie-Hellman
RUN sed -i -e 's/SSL_DHE_RSA_WITH_3DES_EDE_CBC_SHA/TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256/' -e 's/TLS_DHE_RSA_WITH_AES_128_CBC_SHA/TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA/' -e 's/TLS_DHE_DSS_WITH_AES_128_CBC_SHA/TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384/' -e 's/SSL_DHE_DSS_WITH_3DES_EDE_CBC_SHA/TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA/' $TOMCATCFG

# Prevent daemon helper scripts from making systemd calls
ENV SYSTEMCTL_SKIP_REDIRECT=1

# Fix insecure default webserver settings
RUN omconfig preferences webserver attribute=sslencryption setting=autonegotiate
RUN omconfig preferences webserver attribute=signalgorithm setting=SHA256
RUN omconfig preferences webserver attribute=gennewcert cn=hostname org=OMSA OU=IT location=Docker state=none country=DE keysize=2048 webserverrestart=true

# Clean cache to minimize container image
RUN yum clean all

# Restart application to ensure a clean start
CMD srvadmin-services.sh restart && snmpd && tail -f /opt/dell/srvadmin/var/log/openmanage/dcsys64.xml
