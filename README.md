# mesos-edge

![Tested](https://img.shields.io/badge/tested%20with-docker--compose%201.27.4%20-green)

## How to deploy

### Requirements

1. docker engine and compose installed
2. Client registered in IAM with at least the following scopes: openid, profile, email
2. SSL host certificate available

### Step 0: Download this repo

````
git clone https://github.com/iotwins-demo/mesos-edge.git
cd mesos-edge
````

### Step 1: Configuration

1. Copy the host certificate and private key in the `conf/certs` directory

> :warning: the filename of the files *must* be: `cert.pem` for the certificate and `privkey.pem` for the private key

2. Edit the enviroment variables file `conf/env` and provide the needed parameter values:

````
# Provide proper values for the following environment variables

# Set the dns name of the host
export SERVER_NAME=

# Set the url of the IAM and the id and secret of the client to be used
export IAM_URL=
export IAM_CLIENT_ID=
export IAM_CLIENT_SECRET=

# Set a password for crypto purposes, this is used for:
# - encryption of the (temporary) state cookie
# - encryption of cache entries, that may include the session cookie
export CRYPTO_PASSPHRASE=

# Set the IP of the host where the services are running
export DOCKER_HOST_IP=

# You can leave the following line as is
export HAPROXY_SSL_CERT=$(cat conf/certs/cert.pem conf/certs/privkey.pem)
````

3. Load the environment variables

````
source ./conf/env
````

4. Create the apache openidc configuration file launching the following `envsubst` command:

> :warning: if `envsubst` command is not found you may have to install the package `gettext-base`

````
envsubst < conf/httpd-openidc.conf.template > conf/httpd-openidc.conf
````


### Step 2: Build

````
docker-compose build
````


### Step 3: Start services

````
docker-compose up -d
````

The expected output is the following:

````
docker-compose ps
          Name                        Command               State   Ports
-------------------------------------------------------------------------
mesos-edge_chronos_1       /entrypoint.sh                   Up
mesos-edge_marathon_1      /entrypoint.sh marathon          Up
mesos-edge_marathon_lb_1   tini -g -- /marathon-lb/ru ...   Up
mesos-edge_mesosmaster_1   /entrypoint.sh /usr/sbin/m ...   Up
mesos-edge_mesosslave_1    /entrypoint.sh /usr/sbin/m ...   Up
mesos-edge_proxy_1         httpd-foreground                 Up
mesos-edge_zookeeper_1     /entrypoint.sh /usr/share/ ...   Up
````

## Endpoints

The following table shows the list of services and endpoints that are deployed on the host. The ports marked as `Internal` should be kept closed to the external world, whereas those marked as `External` must be open to outside.

| Service  | Port |  Network rule |
| ------------- | ------------- | -----------| 
| Mesos  | master: 5050 (http) <br> nodes: 50501 (http)  | Internal |
| Marathon  | 8080 (http)  | Internal |
| Chronos | 4040 (http) | Internal |
| Apache reverse proxy with OIDC Auth | 443 (https) | External (*) |
| Marathon-LB | 10000-10100 | External (**) |

(*) this endpoint is used by the PaaS Orchestrator to submit tasks via REST APIs. The requests are authenticated through IAM tokens

(**) this endpoint is used to expose the services deployed on Marathon. 
