FROM debian:9

#Install openjdk
RUN apt-get update && apt-get install -y \
      openjdk-8-jdk git maven

#Add a start script for starting the container
ADD start.sh /root/start.sh

#Install MariaDB
RUN apt-get install mariadb-server mariadb-client -y

#Make input/output directories
RUN mkdir /root/datasets && mkdir /root/pipelines && mkdir /root/csv && \
    chmod 777 /root/datasets && chmod 777 /root/pipelines && chmod 777 /root/csv && \
    chmod 755 /root/start.sh

#Download && configure the software
RUN cd /root && git clone https://github.com/sing-group/strep.git && \
    sed -i.bak -e "s/^dataset\.storage.*/dataset.storage=\/root\/datasets\//g" /root/strep/src/main/resources/application.properties && \
    sed -i.bak -e "s/^pipeline\.storage.*/pipeline.storage=\/root\/pipelines\//g" /root/strep/src/main/resources/application.properties && \
    sed -i.bak -e "s/^csv\.storage.*/csv.storage=\/root\/csv\//g" /root/strep/src/main/resources/application.properties && \
    cd /root && git clone https://github.com/sing-group/strep_service.git && \
    sed -i.bak -e "s/^datasetStorage.*/datasetStorage=\/root\/datasets\//g" /root/strep_service/src/main/resources/service.properties && \
    sed -i.bak -e "s/^pipelineStorage.*/pipelineStorage=\/root\/pipelines\//g" /root/strep_service/src/main/resources/service.properties && \
    sed -i.bak -e "s/^outputStorage.*/outputStorage=\/root\/csv\//g" /root/strep_service/src/main/resources/service.properties && \
    mkdir /root/strep_service/resources && mkdir /root/strep_service/config

#Add NLPA configurations and required files
ADD resources /root/strep_service/resources/
ADD config /root/strep_service/config/

#Build strep
RUN mysqld_safe --bind-address=0.0.0.0 & sleep 15 && \
    mysqladmin create strep && \
    #echo "CREATE USER 'springuser'@'%' IDENTIFIED WITH mysql_native_password BY 'springpassword'; GRANT ALL PRIVILEGES on *.* TO 'springuser'@'%' WITH GRANT OPTION; flush privileges; " | mysql && \
    echo "CREATE USER 'springuser'@'%' IDENTIFIED BY 'springpassword'; GRANT ALL PRIVILEGES on strep.* TO 'springuser'@'%'; flush privileges; " | mysql && \
    cd /root/strep && git pull && mvn clean package && mysqladmin shutdown && cd .. 

#Build strep_service
RUN cd /root/strep_service && git pull && \
    mvn clean package && mkdir plugins && cp target/lib/nlpa* plugins/ && \
    cd plugins && jar xf nlpa-* META-INF/services/org.bdp4j.pipe.Pipe && \
    sed -i -E "/^.*(NERFromStringBufferPipe|ComputePolarityFromStringBufferPipe|ComputePolarityTBWSFromStringBuffer|StoreTweetLangPipe)$/d" \
    META-INF/services/org.bdp4j.pipe.Pipe && jar uf nlpa-* META-INF/services/org.bdp4j.pipe.Pipe && rm -rf META-INF && cd .. && \
    cd ..

#Clean the maven library
RUN rm -rf /root/.m2/repository/*

ENTRYPOINT /root/start.sh

EXPOSE 8080 8443