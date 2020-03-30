# Refactor Udagram Monolithic Application Into Microservices
This project is to refactor "Udagram - photo sharing" Monolith application. The aim is to divide the application into smaller microservices. Each microservice needs to run in a separate Docker container. The docker containers needs managed by using the Kubernetes cluster. 
It is aimed at demonstrating independently scaling-up an application, release, and deploying the project using Kubernetes, and TravisCI.

The project has the following services(4):
1. backend-user service
2. backend-feed service
3. frontend service
4. reverseproxy service

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

Install or set up an account on the platforms below to successfully install the project
1. [Install Node and NPM](https://nodejs.org/en/download/)
2. [Installing Ionic Cli](https://ionicframework.com/docs/installation/cli)
3. [Create an account on Amazon Web Services](https://portal.aws.amazon.com/billing/signup#/)
    i. An RDS resource and AWS S3 resource will be setup in the 'SET UP STAGE - STEP 1'
4. [Download and install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)
5. [Install Docker](https://docs.docker.com/install/)
6. [Create an account on DockerHub](https://hub.docker.com/)
7. Create an account on [TravisCI](https://travis-ci.com/)

Necessary but not required
8. [Postman](https://www.getpostman.com/downloads/)
9. [Postbird](https://github.com/udacity/cloud-developer/blob/master/course-02/exercises/udacity-c2-restapi/udacity-c2-restapi.postman_collection.json)

### STEPS FOR SETTING UP PROJECT AND DEPLOYING ON KUBERNETES

#### SET UP STAGE - STEP 1
1. Git clone the [media-upload-project](https://github.com/jakazzy/media-upload-clone)
2. cd into the various directories(below) and for each run npm install
    udacity-c3-frontend
    udacity-c3-restapi-feed
    udacity-c3-restapi-user
3. Set up an RDS and AWS S3 Bucket on Amazon Web Services
4. Create  a .profile file in the Home Directory(if not exists) this will contain user-specific variables. 
5. Copy this path variable and add it to the
```
export PATH=$PATH:/usr/local/mysql/bin/
```
6. Add the values for the following env. variables

```
export POSTGRESS_USERNAME=myusername;
export POSTGRESS_PASSWORD=mypassword;
export POSTGRESS_DATABASE=postgres;
export POSTGRESS_HOST=udagramdemo.abc4def.us-east-2.rds.amazonaws.com;
export AWS_REGION=us-east-2;
export AWS_PROFILE=default;
export AWS_MEDIA_BUCKET=udagramdemo;
export JWT_SECRET=helloworld;
```
After adding the env variables, run 
```
source ~/.profile
```
#### CREATING DOCKER IMAGES - STEP 2

1. Create a docker image with the command
```
docker build -t <your_dockerhub_username_lowercase>/<image_name> . 
```

2. Login in DockerHub via CLI and Push the images after creating them to DockerHub(image registry) using
```
docker push <your_dockerhub_username_lowercase>/<image_name>
```
3. You can subsequently run the image with 

```
docker run --rm --publish 8080:8080 -v $HOME/.aws:/root/.aws --env POSTGRESS_HOST=$POSTGRESS_HOST --env POSTGRESS_USERNAME=$POSTGRESS_USERNAME --env POSTGRESS_PASSWORD=$POSTGRESS_PASSWORD --env POSTGRESS_DB=$POSTGRESS_DB --env AWS_REGION=$AWS_REGION --env AWS_PROFILE=$AWS_PROFILE --env AWS_BUCKET=$AWS_BUCKET --env JWT_SECRET=$JWT_SECRET --name feed <your_dockerhub_username_lowercase>/udacity-restapi-feed
```
4. Remove running containers with

```
docker container ls
docker container kill <container_name>
docker container prune
```
5. cd into udacity-c3-deployment/docker and run
```
docker-compose -f docker-compose-build.yaml build --parallel
```
inorder to build the images for the services

6. Run the application with 
```
docker-compose up
```
 and verify if the application is working successfully by navigating to:

 ```
http://localhost:8100/
 ```

 Stop the containers with:
 ```
docker-compose down
 ```

 #### SET UP KUBERNETES , DEPLOYMENTS, SERVICES - STEP 3

 1. Install Kubernetes on AWS cluster using this [resource](https://github.com/kubermatic/kubeone/blob/v0.11.0/docs/quickstart-aws.md) 

 2. Configure application by running the following

            ```
            kubectl apply -f env-secret.yaml
            kubectl apply -f env-secret.yaml
            ```

3. Create services with

            ```
            kubectl apply -f <name_of_file>
            ```

4. Using port forwarding, run the application 

            ```
            k port-forward service/reverseproxy 8080:8080
            ```
            ```
            k port-forward service/frontend 8100:8100
            ```


#### SET UP CONTINUOUS INTEGRATION AND CONTINUOUS DEPLOYMENT WITH TRAVISCI - STEP 4

1. Add TravisCI as an application on GitHub,then add (your repository) repository.

[follow the set up here](https://docs.travis-ci.com/user/migrate/legacy-services-to-github-apps-migration-guide/)