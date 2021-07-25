[![CircleCI](https://circleci.com/gh/haiwu/devops-capstone/tree/main.svg?style=svg)](https://circleci.com/gh/haiwu/devops-capstone/tree/main)

## Capstone Project for Udacity Cloud DevOps Nanodegree program

## Project Overview
This is the Capstone project for Udacity Cloud DevOPS Nanodegree program. It covers the following: 
  * Working in AWS
  * Using Circle CI to implement Continuous Integration and Continuous Deployment
  * Building pipelines
  * Working with CircleCI aws-eks orb (which uses eksctl, which uses CloudFormation) to deploy AWS EKS cluster
  * Building Docker containers in pipelines
  * nginx hello world application rolling deployment

## CircleCI pipelines
This project has the following pipelines in sequence: 
```
docker/hadolint -> build -> publish-image -> create-cluster -> create-deployment -> aws-eks/update-container-image -> test-cluster
```

## CircleCI DockerHub image repository integration
This project would need to set DOCKERHUB_USERNAME and DOCKERHUB_PASS environment variables in its corresponding CircleCI project settings. 

## CircleCI AWS integration
This project would need to set AWS_ACCESS_KEY_ID, AWS_DEFAULT_REGION and AWS_SECRET_ACCESS_KEY environment variables in its corresponding CircleCI project settings. 

## How AWS EKS cluster is created
Here are the relevant CircleCI config.yml lines used to created AWS EKS cluster: 
```
version: 2.1
orbs:
  aws-eks: circleci/aws-eks@1.1.0
...
jobs:
  create-cluster:
    executor: aws-eks/python3
    parameters:
      cluster-name:
        description: |
          AWS EKS cluster name
        type: string
    steps:
      - aws-eks/create-cluster:
          cluster-name: << parameters.cluster-name >>
          node-type: 't2.micro'
          nodes: 2
          show-eksctl-command: true
          zones: 'us-east-1a,us-east-1b,us-east-1c'
...
workflows:
  devops-captone:
    jobs:
       - create-cluster:
          cluster-name: haiwu-helloworld-eks
          requires:
            - publish-image
```

## How rolling deployment is done
Here are the relevant YAML lines to do this: 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: haiwu-nginx-hello-world-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-world
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
        - name: haiwu-hello-world
          image: haiwu/nginx-hello-world:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
``` 
