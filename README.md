# DevOps_EKS

## This repo contains the terraform files to create the infrastructure on AWS for EKS and also the deployment files for
## frontend ,python backend and node backend 

## How to create the EKS cluster using terraform IAAC

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

## Continues delivery using Codebuild


 ### fe.yml contains the deployment ,service file for react based frontend micro-service

 ### be.yml contains the deployment ,service file for Python(flask) based backend micro-service for Uber business logic

 ### nbe.yml contains the deployment, service file for Node based backend micro-service for User Authentication

 ### Now that we have created the EKS cluster on AWS, we can deploy our deployments and services.

 ### buildspec.yml file contains the commands for creating/updating the deployments ,services on EKS cluster


## Load balancing and Load testing in EKS cluster

 ### We will be using Kubernetes Metrics Server as an aggregator of resource usage data in EKS cluster,

 ### metric-component.yml file contains the set of instructions to install the matrix server on EKS cluster

 ### We are using apache bench for putting the load on the backend \
    ```
     ab -n 150000 -c 50 <Backend_Route_to_be_tested> \
     This will create 150000 request with maximum of 50 concurrent requests at a time.
    ```

 ### The below command will show the details about the scale up and scale down of pods in your EKS cluster
    ```bash
    kubectl describe deploy py-deployment
    ```
    
 ### The below command shows how much is the current load and minimum pods and maximum pods running in your cluster
    ```bash
    kubectl get hpa
    ```
 




