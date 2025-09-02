## Overview
This project is a demo implementation of a system that aggregates, stores, and visualizes logs and metrics. The deployment through Docker-Compose provides Elasticsearch for storage, Kibana for visualization, a Fleet cluster consisting of three nodes to manage Elastic Agents, and an Elastic agent configured to collect data from a Linux server. A custom bridge network named "project" has been created to provide isolated interconnection between containers. Multiple mount points and volumes have been set for data persistence. See the description below for more details about each service in Docker-Compose file.

## Description
**Elasticsearch**:<br>
- image => elasticsearch:9.1.0<br>
- ports => Elasticsearch by default uses port 9200 as a connection gateway with external nodes including kibana, fleet, and elastic agents (also port 9300 is used for inter-cluster communication between Elasticsearch nodes). In this project the 9200 port is mapped into the same port on host so the agents from remote systems can access the Elasticsearch container.<br>
- environment => <br>
>discovery.type => when set to single-node, Elasticsearch creates a single-noded cluster<br>
>xpack.security.enabled => enables authentication and blocks annonymous access to Elasticsearch users if set as true<br>
>xpack.security.http.ssl.enabled => enables communication over https<br>
>xpack.security.http.ssl.keystore.path => specifies the path to keystore of the CA<br>
>xpack.security.http.ssl.certificate_authorities => specifies the path to CA<br>
>ELASTIC_PASSWORD => creates a password for "elastic" user in Elasticsearch<br>
>ES_JAVA_OPTS => specifies init and max use of memory<br>
- volumes => <br>

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
