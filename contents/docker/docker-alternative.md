# Docker application deployed on AWS (with CI integration)

---

## Requirements

 - Java 8 (For optional steps)
 - Maven (For optional steps)
 - Git
 - Compatible IDE (IntelliJ or Eclipse for optional steps)
 - Docker CE

---

# The following steps, 1 to 7 are optional

---

## Step 1 (IntelliJ based)
Go to [this repo](https://github.com/jchacana/spring-boot-docker-demo), clone it with your favorite git tool, either a graphical git client or **CLI**

---

## Step 2 (IntelliJ based)

 - Click on **Import project**
![](images/import.png) <!-- .element: height="400px" -->

---

## Step 3 (IntelliJ based)

 - Select **Import project from external model** then **Maven**
 ![](images/import-maven.png) <!-- .element: height="400px" -->

---

## Step 4 (IntelliJ based)

 - Select the root folder for your project (keep the same location as where you cloned)
 - In the next screen, select profile if applies to your local settings (this will depend on your maven settings, which is out of scope)
 ![](images/maven.png) <!-- .element: height="400px" -->

---

## Step 5 (IntelliJ based)

 - Confirm import of module
 ![](images/maven2.png) <!-- .element: height="400px" -->

---

## Step 6 (IntelliJ based)

 - Confirm project name and location. Press **Finish** and wait for project to be fully imported
![](images/project.png) <!-- .element: height="400px" -->

---

## Step 7 (IntelliJ based)
 - Follow README.md file to compile, build and test
 - If you've executed every step, ee now have a local docker image ready to be run and tested

---

# Step 8: Integrate Docker and AWS

---

## Step 8.1 (Optional, assumes AWS CLI and Docker)

Make sure you have AWS CLI and configure your credentials for AWS and Docker

 - `` aws configure ``
 - `` $(aws ecr get-login --no-include-email --region ap-southeast-1) ``
    + This command will store your credentials info in ~/.docker/docker.json

---

## Step 8.2 (assumes AWS already set)
We go and create a new Docker image repository on AWS Elastic Container Registry (ECR) service

 - `` aws ecr create-repository --repository-name <your_username>/spring-boot-docker ``
 - You'll find useful info in the output
![](images/docker-repository.png) <!-- .element: height="250px" -->

---

## Step 9 (Optional. Execute this in case you have followed steps 1 to 7)
We'll re create our image with the data from our recently created repository:
 - Go to **pom.xml** file
    - Replace **docker.image.prefix** value with **repositoryUri** obtained when we created the repository
 - It should look like this
    - `` <docker.image.prefix>019908562201.dkr.ecr.ap-southeast-1.amazonaws.com/<your_username></docker.image.prefix>  ``
 - Re run `` mvn dockerfile:build ``

---

## Step 9.1 (Optional. Execute this in case you have NOT followed steps 1 to 7)
Here we'll pull an image we'll use for the following steps

- Execute: `` docker pull 019908562201.dkr.ecr.ap-southeast-1.amazonaws.com/mitrais/spring-boot-docker:latest  ``
- This will pull a pre-existing image from our AWS Registry

---

## Step 9.2 (Optional. Execute this in case you have NOT followed steps 1 to 7)
Here we'll tag the image in order to be pushed on step 10. User ``<your_username> ``
- Execute `` docker tag  019908562201.dkr.ecr.ap-southeast-1.amazonaws.com/mitrais/spring-boot-docker:latest 019908562201.dkr.ecr.ap-southeast-1.amazonaws.com/<your_username>/spring-boot-docker:latest  ``


---


## Step 10
Now we'll push our image to our AWS ECR repository
- `` docker push 019908562201.dkr.ecr.ap-southeast-1.amazonaws.com/<your_user>/spring-boot-docker:latest ``
![](images/docker-push.png) <!-- .element: height="50px" -->
![](images/docker-push2.png) <!-- .element: height="150px" -->

---

## Step 11
We check on AWS ECR. 
![](images/aws-ecr.png) <!-- .element: height="350px" -->

---

## Step 12
Now, we'll delete our original image and test our newly uploaded image
 - `` docker image ls `` to get our image list. Identify your image, copy the image id
 - `` docker image rm <image_id>`` to delete the image. Add `` -f `` if force is needed

---

## Step 13
Now we'll run our application
- `` docker run -p 8080:8080 019908562201.dkr.ecr.ap-southeast-1.amazonaws.com/<your_user>/spring-boot-docker``

---

## Step 14
Your docker app should be up and running
![](images/spring-docker.png) <!-- .element: height="350px" -->

---

## Step 15
Go to [http://localhost:8080](http://localhost:8080). You should see a **Hello World!** message

---

# Integrate with CI

---

The following section will cover the creation of a CI server on AWS. This server will be OK for testing purposes. For a production-ready, you'll probably want to implement some security measures and more complex workflows.
For that, take a look at this [AWS Jenkins Whitepaper](https://d0.awsstatic.com/whitepapers/DevOps/Jenkins_on_AWS.pdf)

---
## Jenkins CI
![](images/jenkins.png) <!-- .element: height="350px" -->

---
## Setting up a Jenkins instance on AWS
**A couple of assumptions**
 - Previous knowledge and AWS account correctly configured
 - Previous knowledge on how to connect to the instance via SSH and HTTP
 - AWS Security groups and rules correctly configured

---

## Introduction
This material will allow you prepare your own Execution Cluster with a couple of simple commands using CloudFormation. Follow it and you should have a running instance :-). This steps are based on [This tutorial](https://docs.aws.amazon.com/es_es/AWSGettingStartedContinuousDeliveryPipeline/latest/GettingStarted/CICD_Jenkins_Pipeline.html)

---

## Step 1
For this part of the tutorial, we first need to clone the following git repository

`` https://github.com/jicowan/hello-world ``

The files we're interested in are:
 - ecs-cluster.template
 - ecs-jenkins-demo.template

---
## Step 2
Now we make sure to have AWS CLI on our local environment and configure our credentials with:

``aws configure``

---
## Step 3
Remember to retrieve the data from your ECR (Elastic Container Repository). If you don't remember it, use:

`` aws ecr describe-repositories ``

And take note of your repository's data

---
## Step 3.1
Example:

```json {
    "repositories": [
        {
            "registryId": "019908562201", 
            "repositoryName": "mitrais/spring-boot-docker", 
            "repositoryArn": "arn:aws:ecr:ap-southeast-1:019908562201:repository/mitrais/spring-boot-docker", 
            "createdAt": 1538979498.0, 
            "repositoryUri": "019908562201.dkr.ecr.ap-southeast-1.amazonaws.com/mitrais/spring-boot-docker"
        }
    ]
}
```

---
## Step 4
Execute the following command. This will create your first cluster

`` aws cloudformation create-stack --template-body file://ecs-cluster.template --stack-name EcsClusterStack --capabilities CAPABILITY_IAM --tags Key=Name,Value=ECS --region ap-southeast-1 --parameters ParameterKey=KeyName,ParameterValue=Javier-pem ParameterKey=EcsCluster,ParameterValue=getting-started ParameterKey=AsgMaxSize,ParameterValue=2 ``

---

## Step 5
And execute this command to check status

**DON'T EXECUTE STEP 6 UNLESS THIS COMMAND GIVES YOU CREATE_COMPLETE**

`` aws cloudformation describe-stacks --stack-name EcsClusterStack --query 'Stacks[*].[StackId, StackStatus]' ``

---








