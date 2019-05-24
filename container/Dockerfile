FROM centos:7.3.1611
#FROM centos:latest
#RUN yum -y update
RUN yum -y install httpd 
RUN yum -y install php
RUN rm -rf /var/www/html/*
COPY html/ /var/www/html/
CMD ["/usr/sbin/apachectl", "-D", "FOREGROUND"]
