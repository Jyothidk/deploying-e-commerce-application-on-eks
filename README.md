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


## Deploy ALB controller

Add helm repo

```
helm repo add eks https://aws.github.io/eks-charts
```

Update the repo

```
helm repo update eks
```

Install

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
  -n kube-system \
  --set clusterName=demo-cluster-eks-robot-shop \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-west-1 \
  --set vpcId=vpc-0c49b1db8c0b44b8c
```

Verify that the deployments are running.

```
kubectl get deployment -n kube-system aws-load-balancer-controller
```

![5](https://github.com/Jyothidk/deploying-e-commerce-application-on-eks/assets/127189060/e4642e08-419b-457e-bed8-efbe1e411ee2)


## EBS CSI Plugin configuration

The Amazon EBS CSI plugin requires IAM permissions to make calls to AWS APIs on your behalf.

Create an IAM role and attach a policy. AWS maintains an AWS managed policy or you can create your own custom policy. You can create an IAM role and attach the AWS managed policy with the following command. Replace my-cluster with the name of your cluster. The command deploys an AWS CloudFormation stack that creates an IAM role and attaches the IAM policy to it.

```
eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster demo-cluster-eks-robot-shop \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve
```
Add CSI driver addon

```
eksctl create addon --name aws-ebs-csi-driver --cluster demo-cluster-eks-robot-shop --service-account-role-arn arn:aws:iam::<AWS-ACCOUNT-ID>:role/AmazonEKS_EBS_CSI_DriverRole --force
```

![6](https://github.com/Jyothidk/deploying-e-commerce-application-on-eks/assets/127189060/12e70e37-aac4-449f-b51a-7788fdc1de48)









