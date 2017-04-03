# Overview

This repository contains the documentation and infrastructure settings for OCL's GoCD setup. 

[GoCD](https://www.gocd.io/) is an open-source Continous Delivery tool.

Current setup is triggered by commits on [oclapi](https://github.com/OpenConceptLab/oclapi) and [ocl_web](https://github.com/OpenConceptLab/ocl_web) repositories, deploys to [Showcase](https://showcase.openconceptlab.org), [Staging](https://staging.openconceptlab.org) and [Production](https://www.openconceptlab.org) environments on demand once the quality checks are completed.


Currently, GoCD build server is running on the showcase server. The document below contains information on how the pipeline could be carried, i.e GoCD could be installed on a different server easily. 


> It is crucial that this README and the configuration files are kept up to date in case the pipeline is updated in any way.

The two sections below will help a contirbuter understand how a contribution gets deployed to an environment and how GoCD could be moved to a new location.

   1. [Current build pipeline](#current-build-pipeline)
     * [API](#api)
     * [WEB](#web)
   2. [How to setup GoCD on a new server](#how-to-setup-gocd-on-a-new-server)
     * [Setting up the go-server](#how-to-build-the-gocd-server)
     * [Setting up the go-agent(s)](#how-to-build-the-gocd-agent)
  
   
For more help on GoCD, please visit [GoCD documentation](https://docs.gocd.io/current/)

# Current build pipeline

Current pipeline is on https://gocd.openconceptlab.org

It has two pipelines:
   1. [API](#api)
   2. [WEB](#web)
    
## API

[API pipeline](https://gocd.openconceptlab.org/go/admin/templates/OCL_API/general) has the following stages and corresponding jobs under them which contain specific tasks that carry out the stage's goal
  1. Unit Test
  2. Integration Test
  3. Showcase Deploy
  4. E2E tests
  5. Staging Deploy
  6. Importer Perf
  7. Production Deploy
  
  > __IMPORTANT_NOTE__: Environment variables that start with __OCL__ are meant to be passed on the `ocl_api` container. Without the __OCL__ prefix, they will not be effective.
  
  
### Unit Test
   
This [stage](https://gocd.openconceptlab.org/go/admin/templates/OCL_API/stages/Unit_Test) runs Django unit tests.

### Integration Test
  
This [stage](https://gocd.openconceptlab.org/go/admin/templates/OCL_API/stages/Integration_Test) runs api endpoint tests written in Django

### Showcase Deploy

This [stage](https://gocd.openconceptlab.org/go/admin/templates/OCL_API/stages/Showcase_Deploy) is triggered manually after the integration tests pass. And has the following environment variables:
   * __IP__: Showcase box's IP (showcase.openconceptlab.org)
   * __SETTINGS__: Django settings module
   * __OCL_CONFIG__: Django settings class name 
   * __OCL_SETTINGS__: Django settings
   * __OCL_DATA_ROOT__: Must be left empty so that MongoDB and Solr data is mounted
   * __OCL_AWS_ACCESS_KEY_ID__: This and other AWS variables are used to connect to S3 in order to store exported csv files.
   * __OCL_AWS_SECRET_ACCESS_KEY__
   * __OCL_AWS_STORAGE_BUCKET_NAME__:
   * __OCL_ROOT_PWD__: API admin user's (root) password.
   
### E2E Tests

This [stage](https://gocd.openconceptlab.org/go/admin/templates/OCL_API/stages/E2E) is triggered automatically once the Showcase Deploy stage is completed successfully. It runs UI tests specified under `ocl_web` repo. It has the following environment variable:
   * __IP__: Showcase box's IP
   
### Staging Deploy

This [stage](https://gocd.openconceptlab.org/go/admin/templates/OCL_API/stages/Staging_Deploy) is triggered manually after the integration tests pass. And has the following environment variables:
   * __IP__: Staging box's IP (staging.openconceptlab.org)
   * __SETTINGS__: Django settings module
   * __OCL_CONFIG__: Django settings class name 
   * __OCL_SETTINGS__: Django settings
   * __OCL_DATA_ROOT__: Must be left empty so that MongoDB and Solr data is mounted
   * __OCL_AWS_ACCESS_KEY_ID__: This and other AWS variables are used to connect to S3 in order to store exported csv files.
   * __OCL_AWS_SECRET_ACCESS_KEY__
   * __OCL_AWS_STORAGE_BUCKET_NAME__:
   * __OCL_ROOT_PWD__: API admin user's (root) password.
   * __OCL_NEW_RELIC_API_KEY__: New relic API key, must be updated if NewRelic API key changes.
   
### Importer Perf

This [stage](https://gocd.openconceptlab.org/go/admin/templates/OCL_API/stages/Importer_Perf/) is manually triggered, staging deployment stage must be successful as a prerequisite. It runs a few import jobs on the go-agent container and expects them to complete under 120 seconds. Has two environment variables:
   * __SOLR_ROOT__
   * __SOLR_HOME__
   
### Production Deploy

This [stage](https://gocd.openconceptlab.org/go/admin/templates/OCL_API/stages/Production_Deploy/) is triggered manually. It deploys a given version of the codebase to the production server. __Could be done by anybody who has admin access to the pipeline__. It has the following environment variables:
   * __IP__: Production box's IP (www.openconceptlab.org)
   * __SETTINGS__: Django settings module
   * __OCL_CONFIG__: Django settings class name 
   * __OCL_SETTINGS__: Django settings
   * __OCL_DATA_ROOT__: Must be left empty so that MongoDB and Solr data is mounted
   * __OCL_AWS_ACCESS_KEY_ID__: This and other AWS variables are used to connect to S3 in order to store exported csv files.
   * __OCL_AWS_SECRET_ACCESS_KEY__
   * __OCL_AWS_STORAGE_BUCKET_NAME__:
   * __OCL_ROOT_PWD__: API admin user's (root) password.
   * __OCL_NEW_RELIC_API_KEY__: New relic API key, must be updated if NewRelic API key changes.
   * __DISABLE_VALIDATION__: Disable any custom schema validation (should be False)
   
## WEB

[WEB pipeline](https://gocd.openconceptlab.org/go/admin/pipelines/WEB/general) is fed by the `ocl_web` repository and deploys the UI to various environments. It has the following stages:

   1. Unit Test
   2. Showcase Deploy
   3. E2E Tests
   4. Staging Deploy
   5. Production Deploy
   
 ### Unit Test
 
 This [stage](https://gocd.openconceptlab.org/go/admin/pipelines/WEB/stages/Unit_Test_Job/) run WEB's unit tests
 
### Showcase Deploy
 
This [stage](https://gocd.openconceptlab.org/go/admin/pipelines/WEB/stages/Showcase_Deploy/) is triggered manually. It has the following environment variables:
   * __IP__: Showcase box's IP 
   * __ENV__: Showcase 
   * __PORT__: Port where UI server will be run
   * __TOKEN__: API admin user's API key
    
### E2E Tests

This [stage](https://gocd.openconceptlab.org/go/admin/pipelines/WEB/stages/E2E_Tests) runs UI acceptance tests once the code is deployed to the Showcase environment. It has the following environment variables:
   * __IP__: Showcase box's IP
   
### Staging Deploy

This [stage](https://gocd.openconceptlab.org/go/admin/pipelines/WEB/stages/Staging_Deploy/) is triggered manually, it deploys UI to the Staging environment. It has the following environment variables:
   * __IP__: Staging box's IP 
   * __ENV__: Staging 
   * __PORT__: Port where UI server will be run
   * __TOKEN__: API admin user's API key
   
### Production Deploy

This [stage](https://gocd.openconceptlab.org/go/admin/pipelines/WEB/stages/Production_Deploy/) is triggered manually, it deploys UI to the Production environment. __Could be done by anybody who has admin access to the pipeline__. Has the following environment variables:
   * __IP__: Production box's IP 
   * __ENV__: Production 
   * __PORT__: Port where UI server will be run
   * __TOKEN__: API admin user's API key
   
# How to setup GoCD on a new server

GoCD consists of a GoCD server and one or more GoCD agents. More agents mean more workpower and that many pipelines or stages could be run in parallel.

### How to build the GoCD Server

GoCD server could be install using the latest official docker image for the go-server and running the container as below:

   1. Install docker-engine
   2. ``` docker pull gocd/gocd-server:17.2.0 ```
   3. ``` docker run -d -p 8153:8153 -p 8154:8154 --name go-server --hostname go-server go-server ```
   4. Once the container is up, please go to https://gocd.openconceptlab.org and follow instructions on how to a backup of the current system on https://docs.gocd.io/current/advanced_usage/one_click_backup.html
   5. Follow the instructions on the same document to restore from the GoCD backup.
   6. Visit https://\<SERVER\>:8153 to see whether everything is in place

### How to build the GoCD Agent

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
   6. Visit __AGENTS__ tab on your new GoCD server to see the agents connected successfully to the server.
   7. Should be good to go!
   









