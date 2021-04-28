# DevOps Project Part-2

## This repository contains the terraform files to create the infrastructure on AWS for EKS and also the deployment files for frontend ,python backend and node backend 

## How to create AWS Infrastructure along with EKS cluster using terraform IAAC(Infrastructure as Code)

 1. Initialize terraform repo
    ```bash
    terraform init
    ```
 2. Plan the terraform repo compares the changes with cloud
    ```bash
    terraform plan
    ```
 3. Creates the EKS cluster on AWS
    ```bash
    terraform apply
    ```
 4. terraform output kubeconfig> ~/.kube/config-terraform-eks-demo\
      (After this edit config-terraform-eks-demo and remove EOT/spaces)
 5. aws eks update-kubeconfig --name terraform-eks-demo
 6. terraform output config_map_aws_auth > config-map-aws-auth.yml\
      (After this edit yaml remove EOT and spaces)
 7. Add below entry of rolearn to config-map-aws-auth file which will give access for automatic deployment to codebuild
    ```bash
    - rolearn: arn:aws:iam::<AWS-ACCOUNT-ID>:role/CodeBuildKubectlRole
      username: build
      groups:
        - system:masters
    ```
 8. kubectl apply -f config-map-aws-auth.yml\
      (This will output configmap/aws-auth created)    
 
 9. Now you can access the EKS cluster in your local machine.

## Create the CodeBuildKubectlRole role for CD pipeline using https://github.com/vrdhoke/UberCICDRolesPolicies/blob/master/CDPolicies.png policies

## Create Continues Delivery Pipeline(CD Pipeline,one time setup)

Set up the Code Build Pipeline for Continues Delivery by following the below steps:

   1. Go to AWS Console, navigate to the CodeBuild dashboard and click Create Build project.
   2. Enter  Name: Uber-Codebuild-CD
   3. Description: Deploys the ECR to EKS cluster
   4. Build badge: Check the flag to enable
   5. Use GitHub for the Source provider
   6. Select Connect using OAuth , and click Connect to Github and allow access to your GitHub repo for CODEBUILD-EKS After authenticating, under Repository , select Repository in my GitHub account. Then, add the GitHub repository `DevOps-FinalProject\CODEBUILD-EKS` for this project.
   7. Add source webhook events ,when you check Rebuild every time a code change is pushed to this repository ,any time code
   8. Add enviornment variables 
      - AWS_ACCOUNT_ID with your AWS account ID
      - AWS_DEFAULT_REGION us-west-2
      - AWS_CLUSTER_NAME with terraform-eks-demo
   9. Environment:
      Environment image: use the Managed image
      Operating system: Ubuntu
      Runtime: Standard
      Image: aws/codebuild/standard:4.0
      Image version: Always use the latest image for this runtime version
      Privileged: check the flag
   10. Service role: Existing Role
   Role name: CodeBuildKubectlRole
   11. Under Additional configuration: set the Timeout to 10 minutes
   Buildspec , Artifacts, and Logs: Under Build specifications , 
   select Use a buildspec file
   Skip the Artifacts section ,CloudWatch.
   12. Now that our Codebuild CD pipeline is ready, if developer commit in this repo, the updated ECR images gets deployed to EKS cluster

## Continues delivery using Codebuild

 1. fe.yml contains the deployment ,service file for react based frontend micro-service

 2. be.yml contains the deployment ,service file for Python(flask) based backend micro-service for Uber business logic

 3. nbe.yml contains the deployment, service file for Node based backend micro-service for User Authentication

 4. buildspec.yml file contains the commands for creating/updating the deployments, services on EKS cluster

 5. Now that we have created the EKS cluster on AWS, changes in this repository will deploy ECR images and services over the EKS cluster.


## Load balancing and Load testing in EKS cluster

 1. We will be using Kubernetes Metrics Server as an aggregator of resource usage data in EKS cluster,

 2. metric-component.yml file contains the set of instructions to install the matrix server on EKS cluster

 3. Access the application on Service IP by running the command `kubectl get svc`

 4. We are using apache bench for putting the load on the backend(Python and Node)
    ```bash
     ab -n 150000 -c 100 <Load-balancer-DNS-of-python>:5000/bus/all
     ab -n 150000 -c 100 <Load-balancer-DNS-of-node>:4000/user/nodetest
    ```
    This will create 150000 request with maximum of 100 concurrent requests at a time.

 5. The below command will show the details about the scale up and scale down of pods in your EKS cluster
    ```bash
     kubectl describe deploy py-deployment
    ```
    
 6. The below command shows how much is the current load and minimum pods and maximum pods running in your cluster
    ```bash
     kubectl get hpa
    ```
 




