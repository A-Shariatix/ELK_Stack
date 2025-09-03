## Overview
This project is a demo implementation of a system that aggregates, stores, and visualizes logs and metrics. The deployment through Docker-Compose provides Elasticsearch for storage, Kibana for visualization, a Fleet cluster consisting of three nodes to manage Elastic Agents, and an Elastic Agent configured to collect data from a Linux server. A custom bridge network named "project" has been created to provide isolated interconnection between containers. Multiple mount points and volumes have been set for data persistence. See the description below for more details about each service in Docker-Compose file.

## Description
**Elasticsearch**:<br>
- image => elasticsearch:9.1.0<br>
- ports => Elasticsearch defaults to using port 9200 for its REST API to enable communication with external clients such as Kibana, Fleet, and Elastic Agents. Port 9300 is reserved for inter-node communication within Elasticsearch cluster. As this deployment uses a single-node cluster, the 9300 port is unused. Port 9200 is mapped into the same port on the host to expose the Elasticsearch service to the remote clients.<br>
- environment => <br>
>**discovery.type** -> when set to single-node, Elasticsearch creates a single-noded cluster<br>
>**xpack.security.enabled** -> when set to true, enables authentication and blocks anonymous access to Elasticsearch users<br>
>**xpack.security.http.ssl.enabled** -> when set to true, HTTPS activates and therefore only encrypted traffic is accepted to Elasticsearch<br>
>**xpack.security.http.ssl.keystore.path** -> specifies the path to the PKCS#12 file that includes the required private key and its corresponding X.509 certificate<br>
>**xpack.security.http.ssl.certificate_authorities** -> specifies the path to the CA's certificate file, which is required to verify the authenticity of certificates presented by external clients during mTLS handshake<br>
>**ELASTIC_PASSWORD** -> sets a password for the built-in "elastic" superuser during the initial bootstrap of a new Elasticsearch node<br>
>**ES_JAVA_OPTS** -> used for setting JVM arguments such as Xms (initial heap size) and Xmx (maximum heap size)<br>
- volumes => there are two defined volumes here; es_config and es_data. es_config makes sure elasticsearch keystore and other configs inside config directory persist. es_data on the other hand, saves the data that has been stored in Elasticsearch. there are also two mount points used to share the certs and the CA. the first one mounts two elasticsearch directories together to make the .p12 and CA files available for Elasticsearch. the second one was made at first place to make sure that the made certs and the CA are mounted to the host for two purposes; using openssl which is not available in the containers and making the certs and the CA available for all services.<br>

**Kibana**:<br>
<br>
**Fleet**:<br>
<br>
**Elastic_Agent**:<br>
<br>

## Installation
1- Install docker on your host if you haven't<br>
2- Run "dockerd&" command in terminal to start docker daemon in the background of your host (if it is not running already)<br>
3- Download the files of this repo into your host (note: make sure that the project's structure on your system is as same as it is on this repo)<br>
4- Define your own configurations inside .env file<br>
5- Run "docker compose up" command on your host<br>
