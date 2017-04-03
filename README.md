# Overview

This repository contains the documentation and infrastructure settings for OCL's GoCD setup. 

[GoCD](https://www.gocd.io/) is an open-source Continous Delivery tool.

Current setup is triggered by commits on [oclapi](https://github.com/OpenConceptLab/oclapi) and [ocl_web](https://github.com/OpenConceptLab/ocl_web) repositories, deploys to [Showcase](https://showcase.openconceptlab.org), [Staging](https://staging.openconceptlab.org) and [Production](https://www.openconceptlab.org) environments on demand once the quality checks are completed.


Currently, GoCD build server is running on the showcase server. The document below contains information on how the pipeline could be carried, i.e GoCD could be installed on a different server easily. 


> It is crucial that this README and the configuration files are kept up to date in case the pipeline is updated in any way.

The two sections below will help a contirbuter understand how a contribution gets deployed to an environment and how GoCD could be moved to a new location.


   1. [How to setup GoCD on a new server](#how-to-setup-gocd-on-a-new-server)
   2. [Current build pipeline](#current-build-pipeline)
   
For more help on GoCD, please visit [GoCD documentation](https://docs.gocd.io/current/)

   
# How to setup GoCD on a new server

GoCD consists of a GoCD server and one or more GoCD agents. More agents mean more workpower and that many pipelines or stages could be run in parallel.

### How to build the GoCD Server?

GoCD server could be install using the latest official docker image for the go-server and running the container as below:

   1. Install docker-engine
   2. ``` docker build -f Dockerfile.go-server -t go-server . ```
   2. ``` docker run -d -p 8153:8153 -p 8154:8154 go-server ```

### How to build GoCD Agent?

This repository includes a Dockerfile that is prepared specifically for OCL, which includes necessary applications and libraries needed for deployment and unit testing. We recommend that you use at least two instances of go-agent:

   1. ``` git clone git@github.com:OpenConceptLab/go-cd.git ```
   2. ``` docker build -f Dockerfile.go-agent -t go-agent .```
   3. ``` docker run -d --name go-agent --hostname go-agent -e GO_SERVER_URL=https://$IP:8154/go go-agent ```
   
Once these steps are completed, SSH connection between the go agent and destination environments (e.g staging, showcase) must be established:

   1. Initiate a bash session within the go-agent container:
   ```sh
    docker exec -it go-agent bash
   ```
   2. Once in the container, switch from `root` user to `go` user:
   ```sh
   su go
   ```
   3. Still in the container, generate an SSH key for `go` user:
   ```sh
   ssh-keygen -t rsa
   ```
   4. Still in the container, copy your key over to the deployment environment:
   ```sh
   ssh-copy-id -i $USER@$IP
   ```
   5. Exit the container: type ```exit``` and hit ENTER









