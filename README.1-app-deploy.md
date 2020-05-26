# Module 1: Deploy the Frontend and Backend Applications 

The **Theme Park** application consists of a **Frontend - [Progressive Web App](https://en.wikipedia.org/wiki/Progressive_web_applications) (PWA)** and **Backend** - in this **Module 1**, you will set up the Frontend and then connect it to the Backend.

![Module 1 - Frontend Architecture](./assets/images/module1-arch.png)

Using [AWS Amplify Console](https://aws.amazon.com/amplify/console/), you will create a **Continuous Deployment** pipeline to publish your Frontend application.

The Backend is a set of **Serverless Micro-Services**, including:

* A **DynamoDB** table containing information about all the rides and attractions throughout the park.
* A **Lambda** function which performs a table scan on the DynamoDB to return all the items.
* An **API Gateway** API which creates a public http endpoint for the Front-end application to query. This invokes the Lambda function to return a list of rides and attractions.

![Module 1 - Backend Architecture](./assets/images/module1-backend-arch.png)

Once you have built the Back-end resources needed, you will update the Front-end application configuration to query the **API Gateway endpoint** and display the information about all the rides and attractions.

:heavy_exclamation_mark: Please ensure you have completed the [Setup Guide](https://modern-apps.aws.job4u.io/en/prerequisites/) first!

## 1. Serverless Frontend

Using [AWS Amplify Console](https://aws.amazon.com/amplify/console/), you will create a **Continuous Deployment** pipeline to publish your Frontend application.

[![amplifybutton](https://oneclick.amplifyapp.com/button.svg)](https://ap-southeast-1.console.aws.amazon.com/amplify/home#/deploy?repo=https://github.com/nnthanh101/sls-app)

The Front-end needs to show details of rides and attractions throughout the park to be useful to our park guests. Once you have built the Back-end after this module our guests will be able to see much more useful information in the application.

## 2. Serverless Backend

* A **[DynamoDB table](https://aws.amazon.com/dynamodb/)** which you will populate with information about all the rides and attractions throughout the park.
* A **[Lambda function](https://aws.amazon.com/lambda/)** which performs a table scan on the DynamoDB to return all the items.
* An **[API Gateway API](https://aws.amazon.com/api-gateway/)** creates a public http endpoint for the front-end application to query. This invokes the Lambda function to return a list of rides and attractions.

> Once you have built the Backend resources needed, you will update the Front-end application configuration to query the **API Gateway endpoint**.

## Deploy the Backend infrastructure

Using [AWS Serverless Application Model (**SAM**)](https://aws.amazon.com/serverless/sam/) to deploy Serverless Infrastructure. **SAM** allows you to specify your application requirements in Code and SAM transforms and expands the SAM syntax into AWS CloudFormation to deploy your application. 

**:white_check_mark: Step-by-step Instructions**

1. Go back to your browser tab with [**Cloud9**](https://console.aws.amazon.com/cloud9) running. Make sure your region is correct.

2. Create a deployment bucket in **S3** with a unique name. SAM will upload its code to the bucket to deploy your application services. You will also store this bucket name as an environment variable `s3_deploy_bucket` which will make it easier to type future deployment commands. In the terminal, execute the following commands which pulls your `ACCOUNT_ID` from the Cloud9 Instance metadata and then creates and displays a unique S3 bucket name:
   
```
# ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
ACCOUNT_ID=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r .accountId)
AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')

s3_deploy_bucket="sam-deploys-${ACCOUNT_ID}"

echo $s3_deploy_bucket
```

3. In the terminal, execute the following commands to create the bucket:
```
aws s3 mb s3://$s3_deploy_bucket
```
This has now created the S3 deployment bucket.

4. Change directory:
```
cd ~/environment/sls-app/sam-ride-controller/
```
5. Use SAM CLI to deploy the first part of the infrastructure by running the following commands:
```
sam package --output-template-file packaged.yaml --s3-bucket $s3_deploy_bucket

sam deploy --template-file packaged.yaml --stack-name sam-ride-times --capabilities CAPABILITY_IAM
```

This will take a few minutes to deploy. You can see the deployment progress in the console. Wait until you see the ``Successfully created/updated stack - sam-ride-times`` confirmation message in the console before continuing.

6. Now, change directory:

```
cd ~/environment/sls-app/sam-app/
```
7. Use SAM CLI to deploy the second part of the infrastructure by running the following commands:
```
sam build

sam package --output-template-file packaged.yaml --s3-bucket $s3_deploy_bucket

sam deploy --template-file packaged.yaml --stack-name sam-sls-backend --capabilities CAPABILITY_IAM
```
This will take a few minutes to deploy. You can see the deployment progress in the console. Wait until you see the ``Successfully created/updated stack - sam-sls-backend`` confirmation message in the console before continuing.

SAM has now used CloudFormation to deploy a stack of backend resources which will be used for the rest of the workshop:
- 2 **Lambda Functions** and a **Lambda Layer**
- 3 **S3 Buckets**
- A **DynamoDB Table**
- A **Cognito UserPool**
- An **AWS IoT** thing
- Several **IAM Roles** and **Policies**.

8. Configure environment variables. 
   
Set a number of environment variables to represent the custom names of resources deployed in your account. These commands use the AWS CLI to retrieve the CloudFormation resource names and then construct the environment variables using Linux string manipulation commands ``grep`` and ``cut``. This makes it easier to type deployment commands in later modules. In the terminal, execute:

```console
AWS_REGION=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone | sed 's/\(.*\)[a-z]/\1/')
FINAL_BUCKET=$(aws cloudformation describe-stack-resource --stack-name sam-sls-backend --logical-resource-id FinalBucket --query "StackResourceDetail.PhysicalResourceId" --output text)
PROCESSING_BUCKET=$(aws cloudformation describe-stack-resource --stack-name sam-sls-backend --logical-resource-id ProcessingBucket --query "StackResourceDetail.PhysicalResourceId" --output text)
UPLOAD_BUCKET=$(aws cloudformation describe-stack-resource --stack-name sam-sls-backend --logical-resource-id UploadBucket --query "StackResourceDetail.PhysicalResourceId" --output text)
DDB_TABLE=$(aws cloudformation describe-stack-resource --stack-name sam-sls-backend --logical-resource-id DynamoDBTable --query "StackResourceDetail.PhysicalResourceId" --output text)
echo $FINAL_BUCKET
echo $PROCESSING_BUCKET
echo $UPLOAD_BUCKET
echo $DDB_TABLE
```

The terminal now looks like this, echoing back all the set environment variables:

SAM has now used CloudFormation to deploy a stack of backend resources which will be used for the rest of the workshop, 2 x Lambda functions and a Lambda Layer, 3 x S3 buckets, a DynamoDBTable, Cognito UserPool, AWS IoT thing and a number of IAM Roles and Policies.

## Populate the DynamoDB Table

DynamoDB is a key-value and document database which we will use to store information about all the rides and attractions throughout the park.

The SAM template created a DynamoDB table for the application. Next, you will fill the DynamoDB table with data about the rides and attractions in the park. You will run a local Node script in this repo to upload the data to DynamoDB.

**:white_check_mark: Step-by-step Instructions**

1. From the Cloud9 console, navigate to the local-app directory:
```
cd ~/environment/sls-app/local-app/
```
2. Install the NPM packages needed:
```
npm install
```
*Ignore any NPM warnings or errors - do not run npm audit*

3. Run the import script:
```
node ./importData.js $AWS_REGION $DDB_TABLE
```