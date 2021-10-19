# Using Elestic Kubernetes Service (EKS)
- Create a Node.js + Express Project
    - USe all required dewpendencies
    - CReate Dockerfile and .dockerignore file
- CReate a docker image
    - docker build . -t eksdemo:v1
- Test the app from the image
    - docker run -p 9070:9070 --name=mycont eksdemo:v1        
- Download the AWS CLI and eksctl
    - Configure the aws configure on the dev maching by providing
        - AccessKeyId, SecretAccessKey , Region, Format
- Make sure that, your subscription has access right to perform operations like 
    - aws configure
    - create ECR
    - cerate EKS       
- CReate a Repository on AWS for Pushing Images
    - The Elastic Container Repository (ECR)   
        - aws ecr create-repository --repository-name [REPOSITORY-NAME] --region [REGION] 
        - e.g.
            - aws ecr create-repository --repository-name eksdemoecr --region us-east-2        
    - Repository details will be as follows

``` javascript
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:us-east-2:[SUBSCIPTION-ID-OF-AWS]:repository/eksdemoecr",
        "registryId": "472804039072",
        "repositoryName": "eksdemoecr",
        "repositoryUri": "[SUBSCIPTION-ID-OF-AWS].dkr.ecr.us-east-2.amazonaws.com/eksdemoecr",
        "createdAt": "2021-10-19T16:29:22+05:30",
        "imageTagMutability": "MUTABLE",
        "imageScanningConfiguration": {
            "scanOnPush": false
        },
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        }
    }
}
```

- Login on the ECR
    - The command is as below
        - aws ecr get-login-password --region [REGIOn-NAME] | docker login --username AWS --password-stdin [SUBSCRIPTION-ID-OF-AWS].dkr.ecr.us-east-2.amazonaws.com/[REPOSITORY-NAME]
        - e.g.
            - aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin [SUBSCIPTION-ID-OF-AWS].dkr.ecr.us-east-2.amazonaws.com/eksdemoecr
- PUSH the DOcker Image from the Local to ECR
    - Tag the Docker Image to ECR-NAME          
        - docker tag [IAMEG-NAME]:[TAG-NAME] [SUBSCRIPTION-ID-OF-AWS].dkr.ecr.[REGION-NAME].amazonaws.com/[REPOSITORY-NAME]:[TAG]      
        - e.g.
            - docker tag eksdemo:v1 [SUBSCIPTION-ID-OF-AWS].dkr.ecr.us-east-2.amazonaws.com/eksdemoecr:v1 
    - Push the newly tagged Image    
        - docker push [SUBSCIPTION-ID-OF-AWS].dkr.ecr.us-east-2.amazonaws.com/eksdemoecr:v1    

- Creating EKS
    - EKS is a CLuster Manager Service provide by AWS for managing Microservices based apps
        - Nodes to Host PODS
        - PODS to Host COntainers
        - COntainer to Host image
    - IN the project add a 'kube' folder and add deplopyment.yml and service.yml in it
    - All Nodes are EC 2 Instances
        - EC 2 Pricing is applied
    - FOr the cluster creation
        - Manage Security Groups
        - Manage the Public and Private Communication across Nodes in the cluster
        - The IAM Role is Mandatory for Cluster Creation
        - Use the 'CloudFormation' Template to CReate the Cluster with 
            - EC2 Configuration        
            - Subnets
            - Public and Private Cominucations
        - USing the Following JSON Schema for the CLuster Creation
            - https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-06-10/amazon-eks-vpc-private-subnets.yaml
            - Go to CloudFormation form AWS Portal
            - CLick on 'Create Stack'
            - Select the 'Template Source' as 'Amazon S3 URL'
                - Paste the Above URL in it 
            - CLick on Next
            - ENter Stack NAme
                - Here we can see the Public and Private     Subnet IPs for CLuster
            - CLick on Next
            - CLick on Next
            - CLick on Create Stack 
                - The Stack will be created
        - The Cloud Formation Stack will be created with the FOllowing Information
            - VPC-ID
                - an Id of VM that will be used by the cluster to manage deployment
            - SecurityGroup
                - Set of Rules for In-Boud and Out-Bound Calls or COmmunications
            - SubnetIds
                - Private and Public Ids for internal and external counications
        - Create a Cluster.yml or Cluster.json file for defining the Stack Information for creating Cluster
            - On Windows Machine put this JSON file in
                - c:\users\[USER-NAME]\.kube\config
            - On Mac and Linux Machine
                - ~/.kube/config                
    - Create a CLuster using Cluster.json using the following command
        - eksctl create cluster -f cluster.json --kubeconfig=c:\users\{user}\.kube\config                               
    - Once the cluster is created set the cutrrent K8S configuration for the newly created cluster
        - aws eks --region us-east-2 update-kubeconfig --name EKS-node-Cluster-b2        
    - deploy the service 
        - kubectl apply -f .\kube\deployment.yml 
        kubectl apply -f .\kube\service.yml       
    - To access the service, add the NodePOrt into the security group    