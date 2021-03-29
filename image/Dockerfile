FROM harbor.aks.au-poc.com/library/softwareag/webmethods-microservicesruntime:10.5.0.1

ADD --chown=1724:1724 *.zip *.md5 /opt/softwareag/IntegrationServer/replicate/autodeploy/
ADD --chown=1724:1724 application.properties /opt/softwareag/IntegrationServer/

EXPOSE 5555 9999 8092 8093 8094
