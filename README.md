# Overview of the Project Tasks

Start with the "Udagram - photo sharing" Monolith application and divide the application into smaller (micro)services. Each microservice must run in a separate Docker container. These containers (and ReplicaSets) have to be managed by using the Kubernetes cluster. Demonstrate the ability to independently scale-up, release, and deploy the project using Kubernetes, and TravisCI.

## Getting started

### Prerequisites
The following tools need to be installed on your machine:

- [Docker](https://www.docker.com/products/docker-desktop)
- [AWS CLI](https://aws.amazon.com/cli/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

Furthermore, you need to have:
- an [Amazon Web Services](https://console.aws.amazon.com) account
- a [DockerHub](https://hub.docker.com/) account
- a [Travis](https://travis-ci.com) account

### Clone the repository

Clone the repository on your local machine:

```
git clone https://github.com/WeiWu0210/kube.git
```

### Create an S3 bucket

The application uses an S3 bucket to store the images so an AWS S3 Bucket needs to be created

#### Permissions

Save the following policy in the Bucket policy editor:

```JSON
{
 "Version": "2012-10-17",
 "Id": "Policy1565786082197",
 "Statement": [
 {
 "Sid": "Stmt1565786073670",
 "Effect": "Allow",
 "Principal": {
 "AWS": "__YOUR_USER_ARN__"
 },
 "Action": [
 "s3:GetObject",
 "s3:PutObject"
 ],
 "Resource": "__YOUR_BUCKET_ARN__/*"
 }
 ]
}
```
Modify the variables `__YOUR_USER_ARN__` and `__YOUR_BUCKET_ARN__` by your own data.

#### CORS configuration

Save the following configuration in the CORS configuration Editor:

```XML
<?xml version="1.0" encoding="UTF-8"?>
 <CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
 <CORSRule>
 <AllowedOrigin>*</AllowedOrigin>
 <AllowedMethod>GET</AllowedMethod>
 <AllowedMethod>POST</AllowedMethod>
 <AllowedMethod>DELETE</AllowedMethod>
 <AllowedMethod>PUT</AllowedMethod>
 <MaxAgeSeconds>3000</MaxAgeSeconds>
 <AllowedHeader>Authorization</AllowedHeader>
 <AllowedHeader>Content-Type</AllowedHeader>
 </CORSRule>
</CORSConfiguration>
```

## Deploy on local

`Docker` is used to start the application on the local environment

The variables below need to be added to your environment such as .bash_profile:

```
export POSTGRESS_USERNAME=
export POSTGRESS_PASSWORD=
export POSTGRESS_DATBASE=
export POSTGRESS_HOST=
export AWS_REGION=
export AWS_PROFILE=
export AWS_MEDIA_BUCKET=
export JWT_SECRET=
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
export DOCKER_USERNAME=
export DOCKER_PASSWORD=
```

Replace the variables `__YOUR_AWS_BUCKET_NAME__`, `__YOUR_AWS_BUCKET_REGION__` and `__YOUR_AWS_PROFILE__` by your own information

Build the images by running:

```
docker-compose -f docker-compose-build.yaml build --parallel
```

Start the application and services:

```
docker-compose up
```

The application is now running at http://localhost:8100

## Deploy on AWS

The application is running in a Kubernetes Cluster on AWS.

### Create a Kubernetes cluster at AWS
https://medium.com/containermind/how-to-create-a-kubernetes-cluster-on-aws-in-few-minutes-89dda10354f4

#### Provision the infrastructure
```
export KOPS_CLUSTER_NAME= AWS_CLUSTER_NAME
export KOPS_STATE_STORE= AWS_S3_BUCKET
kops create secret --name ${KOPS_CLUSTER_NAME} sshpublickey admin -i ~/.ssh/ec2_kube.pub
kops create cluster --node-count=2 --node-size=t3.medium --zones=us-west-1a --name=${KOPS_CLUSTER_NAME} --yes
```
### Create a PostgreSQL Instance

The application is using `PostgreSQL` database to store the feed data.

Create a PostgresSQL instance via Amazon RDS.

Add the ```udagram_common``` VPC security group to your Database instance so the services can access it.

### Build the production images

Build the images by executing:

```
docker-compose -f docker-compose-build.yaml --parallel
```

Push your images to your Docker Hub

```
docker-compose -f docker-compose-build.yaml push
```

### Deploy the Kubernetes pods

For each deployment.yaml in `deployment/k8s` replace the image name by your own Docker Hub name. Example:

Deploy the Kubernetes pods by running

```
  - kubectl apply -f udacity-c3-deployment/k8s/backend-feed-service.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/backend-user-service.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/frontend-service.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/reverseproxy-service.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/aws-secret.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/env-configmap.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/env-secret.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/backend-feed-deployment.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/backend-user-deployment.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/frontend-deployment.yaml
  - kubectl apply -f udacity-c3-deployment/k8s/reverseproxy-deployment.yaml
```

### Deploy a canary version of your application

2 versions of the application can be run on parallel for A/B Testing.

Checkout the V2 branch by:

```
git checkout -b V2
```

Add the tag V2 on the frontend image in your Docker production build file and build it:

```
docker-compose -f __YOUR_DOCKER_BUILD_FILE__ build --parallel
```

Push the V2 frontend image to your Docker Hub

```
docker-compose -f __YOUR_DOCKER_BUILD_FILE__ push
```

Deploy the V2 frontend in Kubernetes

```
kubectl -f deployment/k8s/frontend-canary-deployment.yaml
```
