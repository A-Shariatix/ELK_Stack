## Overview
This project is a demo implementation of a system that aggregates, stores, and visualizes logs and metrics. The Docker Compose deployment provides Elasticsearch for storage, Kibana for visualization, a Fleet server cluster consisting of three nodes to manage Elastic Agents, and an Elastic agent configured to collect data from a Linux server. A custom bridge network named "project" has been created to provide isolated interconnection between containers. Multiple mount points and volumes have been set for data persistence. See the description below for more details.

## Description
**Elasticsearch**:<br>
The elastic container uses elasticsearch:9.1.0 image. It is configured to run in single-node mode. The 9200 port is the default port of Elasticsearch and it is mapped to the same port on host, so it is accessible for the agents outside of docker. <br>
**Kibana**:<br>
The kibana container uses kibana:9.1.0 image. <br>
**Fleet**:<br>
The fleet containers use elastic/elastic-agent:9.1.0 image. <br>
**Elastic_Agent**:<br>
The linux-agent container uses the same image as fleet containers. <br>

