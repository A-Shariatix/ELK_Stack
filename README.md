## Overview
This project provisions a system for aggregating, storing, and visualizing logs and metrics. The Docker Compose deployment includes Elasticsearch for storage, Kibana for visualization, a Fleet cluster of three nodes to manage Elastic Agents, and an Elastic Agent configured to collect data from a Linux server. A custom bridge network named "project" has been created to provide isolated interconnection between containers. Multiple mount points and volumes ensure data persistence. See the description below for more details about each service in Docker-Compose file.

## Description
### **Elasticsearch**:<br>
- #### image:
  elasticsearch:9.1.0<br>
- #### ports:
  Port 9200 on host is mapped to port 9200 (default port) on the Elasticsearch container, exposing the REST API. Port 9300, used for internal cluster communication, is unused in this single-node deployment.<br>
- #### environment:
  Configuration is managed through environment variables for clarity and ease of deployment. Below is a detailed explanation of the environment variables used:<br>
  
  - **discovery.type** -> When set to single-node, allows a node to start successfully without other cluster nodes. If unset, Elasticsearch defaults to its standard discovery process, which expects to form a multi-node cluster and may cause warnings if other nodes are unreachable.<br>
  - **xpack.security.enabled** -> When set to true, enables authentication and blocks anonymous access to Elasticsearch users.<br>
  - **xpack.security.http.ssl.enabled** -> When set to true, enables HTTPS and enforces TLS for all connections to the Elasticsearch REST API.<br>
  - **xpack.security.http.ssl.keystore.path** -> Specifies path to the PKCS#12 file that includes the required private key and its corresponding X.509 certificate.<br>
  - **xpack.security.http.ssl.client_authentication** -> Has three options: required, optional, and none (which is the default one). In this config client authentication level is set to optional, which means the Elasticsearch server will ask for clients' certificates, but it's not necessary for clients to provice certificates. However, if a client provides a certificate, it must be valid.<br>
  - **xpack.security.http.ssl.certificate_authorities** -> Specifies path to the CA certificate file used to verify the authenticity of certificates presented by external clients during TLS handshake. This setting is required for mTLS.<br>
  - **ELASTIC_PASSWORD** -> Sets a password for the built-in "elastic" superuser during the initial bootstrap of a new Elasticsearch node.<br>
  - **ES_JAVA_OPTS** -> Used for setting JVM arguments such as Xms (initial heap size) and Xmx (maximum heap size).<br>
- #### volumes:
  - **es_config** -> Persists contents of /usr/share/elasticsearch/config directory. One of the important files in that directory is elasticsearch.keystore which holds the password for the PKCS#12 file.<br>
  - **es_data** -> Persists Elasticsearch data including indices and data streams in /usr/share/elasticsearch/data directory.<br>
  - **./certs/elasticsearch** -> Mounts Elasticsearch PKCS#12 keystore (containing private key, CA, and public certificate) and the CA from the host to /usr/share/elasticsearch/config/elasticsearch directory in container.<br>
  - **./certs** -> Mounts the host's ./certs directory to provide certificate files to the Elasticsearch container.<br>


### **Kibana**:<br>
- #### image:
  kibana:9.1.0<br>
- #### ports:
  Port 5601 on host is mapped to port 5601 (default port) on Kibana container, exposing Kibana's HTTP interface. This makes Kibana accessible on host browsers.<br>
- #### environment:
  - **ELASTICSEARCH_HOSTS** -> Defines the URL for Kibana to connect to Elasticsearch. Set to https://elastic:9200 to use the service name elastic defined in the Docker Compose network and to enforce a secure HTTPS connection.<br>
  - **ELASTICSEARCH_USERNAME** -> Specifies an Elasticsearch username, which is "kibana_system" in this case. "kibana_system" is a pre-built user designed with required privileges for Kibana.<br>
  - **ELASTICSEARCH_PASSWORD** -> Sets the password for the kibana_system user. This password must be set manually in Elasticsearch and matched here for Kibana to connect successfully.<br>
  - **ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES** -> Specifies path to the CA file that verifies Elasticsearch's signed certificate.<br>
  - **SERVER_SSL_ENABLED** -> When set to true, enables communication over HTTPS and forces every connection to the Kibana server to use TLS.<br>
  - **SERVER_SSL_CERTIFICATE** -> Specifies the path to Kibana's public TLS certificate file. This certificate is presented to clients to authenticate the Kibana server's identity during the TLS handshake.<br>
  - **SERVER_SSL_KEY** -> Specifies the path to Kibana's private key file. This key is used to prove ownership of the public certificate during the TLS handshake.<br> 
  - **SERVER_SSL_KEY_PASSPHRASE** -> Sets the password of the private key file, to make it accessible for Kibana. This password gets set when private key is getting extracted from the PKCS#12 file created by elasticsearch-certutil<br>
  - **XPACK_ENCRYPTEDSAVEDOBJECTS_ENCRYPTIONKEY** -> Sets an encryption key for securing Kibana's saved objects when they are sent to or received from Elasticsearch. In this project, it is a random key generated by openSSL.<br>
  
- #### volumes:
  - **./certs/kibana:** -> Mounts Kibana public certificate, private key, and the CA from the host to /usr/share/kibana/config/kibana directory in container.<br>


### **Fleet**:<br>
- #### image:
  elastic/elastic-agent:9.1.0<br>
- #### ports:
  Port 8220 on host is mapped to port 8220 (default port when the agent is configured to act as a Fleet server) on Fleet containers, exposing Fleet server's API for external agents. <br>
- #### environment:
  - **ELASTICSEARCH_HOST** -> Defines the URL for Fleet server to connect to Elasticsearch. Set to https://elastic:9200 to use the service name elastic defined in the Docker Compose network and to enforce a secure HTTPS connection.<br>
  - **FLEET_SERVER_ELASTICSEARCH_CA_TRUSTED_FINGERPRINT** -> Specifies the Elasticsearch trusted fingerprint. Used instead of the CA file. To use the CA file, "FLEET_SERVER_ELASTICSEARCH_CA" variable can be used.<br>
  - **FLEET_SERVER_ENABLE** -> When set to 1, sets Fleet server up on the Elastic Agent.<br>
  - **FLEET_SERVER_SERVICE_TOKEN** -> Specifies the generated service token by Elasticsearch. This token gets used by a service account to make the connection possible between Elasticsearch and Fleet server.<br>
  - **FLEET_SERVER_POLICY_ID** -> Specifies the created policy's ID. The policy can be created through Fleet API or Fleet UI on Kibana.<br>
  - **FLEET_SERVER_HOST** -> Defines the IP address that hosts the Fleet server in a container. The default value is 127.0.0.1.<br>
  - **FLEET_SERVER_PORT** -> Defines the port that is bound to the mentioned IP address. The default value is 8220.<br>
  - **FLEET_URL** -> Specifies an endpoint that the agents will use to connect to this Fleet Server.<br>
  - **FLEET_CA** -> Specifies path to the CA file.<br>
  - **FLEET_SERVER_TIMEOUT** -> Defines how long the Fleet server waits for Elasticsearch to start working. defaults to 2 minutes.<br>
  - **FLEET_SERVER_CERT** -> Specifies path to Fleet server's public certificate file.<br>
  - **FLEET_SERVER_CERT_KEY** -> Specifies path to Fleet server's private key file.<br>
  - **FLEET_SERVER_SSL_KEY_PASSPHRASE** -> Sets the password of private key file.<br>
- #### volumes:
  - **./certs/fleet:** -> Mounts Fleet public certificate, private key, and the CA from the host to /usr/share/elastic-agent/fleet directory in container.<br>


### **Elastic_Agent**:<br>
- #### image:
  elastic/elastic-agent:9.1.0<br>
- #### environment:
  - **FLEET_ENROLL** -> When set to true, enables agent enrollment into Fleet server.<br>
  - **FLEET_CA** -> Specifies path to the CA file.<br>
  - **FLEET_URL** -> Specifies the target Fleet server URL for initial enrollment and furthermore management.<br>
  - **FLEET_ENROLLMENT_TOKEN** -> The Fleet server's enrollment token used to enroll the agent at first place.<br>
- #### volumes:
  - **/etc/localtime** -> Synchronizes container's localtime with the host.<br>
  - **/etc/timezone** -> Synchronizes container's timezone with the host.<br>
  - **./certs/fleet/ca.crt** -> Mounts the CA file to /usr/share/elastic-agent/linux-agent/ca.crt file in container<br>
  - **/var/lib/elastic-agent** -> Shares the agent's state data (like policy, enrollment state, and ...) from the container /var/lib/elastic-agent directory into host, making these data persist across container restarts.<br>
  - **/var/log** -> Mounts contents of /var/log from the host into the container in /hostfs/var/log path. This enables the agent to track the host's logs from inside the container.<br>
  - **/proc** -> Mounts contents of /proc from the host into the container in /hostfs/proc path. This makes the agent capable of montioring the host's metrics and kernel-level data from inside the container.<br>
  - **/sys** -> Mounts contents of /sys from the host into the container in /hostfs/sys path. This will let the agent to monitor the host's hardware-level data from inside the container.<br>

## Installation
1. Install and run docker (https://docs.docker.com/engine/install/).<br>
2. Download the files of this repo into your host. you can also use your own certificates by replacing them with the current ones (note: make sure that the certs directory is on the same path as docker-compose.yml is).<br>
3. Add Elasticsearch private key's password to the keystore using "./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password" command inside the elastic container.<br>
4. Change password of the "kibana_system" bulit-in user from inside the elastic container using "bin/elasticsearch-reset-password -u kibana_system -i" command (if you remove the -i, a random password will be generated). The point of making this step, is to make sure that Kibana can use the mentioned user to connect to Elasticsearch. this password can't get set through docker-comopse.yml (only the "elastic" user's password can be set through config).<br>
5. Define your own configurations in the .env file and place it into the same path as the docker-compose.yml. There is a also a ready to use .env file in files section.<br>
6. Run "docker compose up" command on your host to deploy the services (make sure your host serves at least 8GB or more free memory).<br>
7. To track auditd logs, install auditd integration through FLEET UI.
