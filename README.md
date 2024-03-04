# Deploying E Commerce Robotshop application on EKS cluster

Robot shop is a sample popular Microservices application. It is owned by Instana which is acquired by IBM. They use this project in their product developments like instana APM tool and other products. 

Prerequisites: Install kubectl, eksctl and aws cli and configure it on your system.

## Install EKS 

```
eksctl create cluster --name demo-cluster-eks-robot-shop --region us-west-1
```

![1](https://github.com/Jyothidk/deploying-e-commerce-application-on-eks/assets/127189060/084ff486-6f8b-4f94-b15e-085623384bb5)

## Configure IAM OIDC provider

```
export cluster_name=demo-cluster-eks-robot-shop
```
```
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5) 
```
Check if there is an IAM OIDC provider configured already
```
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
```
If not, run the below command
```
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```

![2](https://github.com/Jyothidk/deploying-e-commerce-application-on-eks/assets/127189060/40e2cfe3-0eac-4b0e-826b-58665507e1cd)

## How to setup alb add on

Download IAM policy

```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```

Create IAM Policy

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

![3](https://github.com/Jyothidk/deploying-e-commerce-application-on-eks/assets/127189060/4394ee95-9bfe-4336-bb37-3453ee79039b)

Create IAM Role

```
eksctl create iamserviceaccount \
  --cluster=demo-cluster-eks-robot-shop \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```
![y](https://github.com/Jyothidk/deploying-e-commerce-application-on-eks/assets/127189060/7dd1ef92-53c2-41cb-868e-a159451e2e0d)





