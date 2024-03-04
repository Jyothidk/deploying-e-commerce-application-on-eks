# Deploying E Commerce Robotshop application on EKS cluster

Robot shop is a sample popular Microservices application. It is owned by Instana which is acquired by IBM. They use this project in their product developments like instana APM tool and other products. 

Prerequisites: Install kubectl, eksctl and aws cli and configure it on your system.

## Install EKS 

```
eksctl create cluster --name demo-cluster-eks-robot-shop --region us-west-1
```

![1](https://github.com/Jyothidk/deploying-e-commerce-application-on-eks/assets/127189060/084ff486-6f8b-4f94-b15e-085623384bb5)

