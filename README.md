## Overview
This project is a demo implementation of a system that aggregates, stores, and visualizes logs and metrics. The deployment through Docker-Compose provides Elasticsearch for storage, Kibana for visualization, a Fleet cluster consisting of three nodes to manage Elastic Agents, and an Elastic Agent configured to collect data from a Linux server. A custom bridge network named "project" has been created to provide isolated interconnection between containers. Multiple mount points and volumes have been set for data persistence. See the description below for more details about each service in Docker-Compose file.

## Description
### **Elasticsearch**:<br>
- image => elasticsearch:9.1.0<br>
- ports => Elasticsearch defaults to using port 9200 for its REST API to enable communication with external clients such as Kibana, Fleet, and Elastic Agents. Port 9300 is reserved for inter-node communication within Elasticsearch cluster. As this deployment uses a single-node cluster, the 9300 port is unused. Port 9200 is mapped into the same port on the host to expose the Elasticsearch service to the remote clients.<br>
- environment => There are two ways to configure the services: using config files and mounting them via volumes section, or defining configuration variables in the environment section (which is used in this project). Below is a detailed explanation of used environment variables:<br>
>**discovery.type** -> when set to single-node, Elasticsearch creates a single-noded cluster<br>
>**xpack.security.enabled** -> when set to true, enables authentication and blocks anonymous access to Elasticsearch users<br>
>**xpack.security.http.ssl.enabled** -> when set to true, HTTPS activates and therefore only encrypted traffic is accepted to Elasticsearch<br>
>**xpack.security.http.ssl.keystore.path** -> specifies the path to the PKCS#12 file that includes the required private key and its corresponding X.509 certificate<br>
>**xpack.security.http.ssl.certificate_authorities** -> specifies the path to the CA's certificate file, which is required to verify the authenticity of certificates presented by external clients during mTLS handshake<br>
>**ELASTIC_PASSWORD** -> sets a password for the built-in "elastic" superuser during the initial bootstrap of a new Elasticsearch node<br>
>**ES_JAVA_OPTS** -> used for setting JVM arguments such as Xms (initial heap size) and Xmx (maximum heap size)<br>
- volumes => Docker provides two methods for data persistence: named volumes and bind mounts. Two volumes are exclusively used for Elasticsearch in this project, named es_config and es_data. There are also two mount points used for sharing data between the host and Elasticsearch container. Check the below explanation for more details:<br>
>**es_config** -> <br>
>**es_data** -> <br>
>**bind_mount_1** -> <br>
>**bind_mount_2** -> <br>
### **Kibana**:<br>
- image => <br>
- ports => <br>
- environment => <br>
- volumes => <br>

### **Fleet**:<br>
- image => <br>
- ports => <br>
- environment => <br>
- volumes => <br>

### **Elastic_Agent**:<br>
- image => <br>
- ports => <br>
- environment => <br>
- volumes => <br>

## Installation
1- Install docker on your host if you haven't<br>
2- Run "dockerd&" command in terminal to start docker daemon in the background of your host (if it is not running already)<br>
3- Download the files of this repo into your host (note: make sure that the project's structure on your system is as same as it is on this repo)<br>
4- Define your own configurations inside .env file<br>
5- Run "docker compose up" command on your host<br>
