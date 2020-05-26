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
   
```bash
# ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
ACCOUNT_ID=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r .accountId)
AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')

s3_deploy_bucket="sam-deploys-${ACCOUNT_ID}"

echo $s3_deploy_bucket
```

3. In the terminal, execute the following commands to create the bucket:

```bash
aws s3 mb s3://$s3_deploy_bucket
```

This has now created the S3 deployment bucket.

4. Change directory:

```bash
cd ~/environment/sls-app/sam-ride-controller/
```

5. Use SAM CLI to deploy the first part of the infrastructure by running the following commands:

```bash
sam package --output-template-file packaged.yaml --s3-bucket $s3_deploy_bucket

sam deploy --template-file packaged.yaml --stack-name sam-ride-times --capabilities CAPABILITY_IAM
```

This will take a few minutes to deploy. You can see the deployment progress in the console. Wait until you see the ``Successfully created/updated stack - sam-ride-times`` confirmation message in the console before continuing.

6. Now, change directory:

```bash
cd ~/environment/sls-app/sam-app/
```

7. Use SAM CLI to deploy the second part of the infrastructure by running the following commands:

```bash
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

```bash
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

SAM has now used CloudFormation to deploy a stack of backend resources which will be used for the rest of the workshop, 2 x Lambda functions and a Lambda Layer, 3 x S3 buckets, a DynamoDBTable, Cognito UserPool, AWS IoT thing and a number of IAM Roles and Policies.

## Populate the DynamoDB Table

DynamoDB is a key-value and document database which we will use to store information about all the rides and attractions throughout the park.

The SAM template created a DynamoDB table for the application. Next, you will fill the DynamoDB table with data about the rides and attractions in the park. You will run a local Node script in this repo to upload the data to DynamoDB.

**:white_check_mark: Step-by-step Instructions**

1. From the Cloud9 console, navigate to the local-app directory:

```bash
cd ~/environment/sls-app/local-app/
```

2. Install the NPM packages needed:

```bash
npm install
```

*Ignore any NPM warnings or errors - do not run npm audit*

3. Run the import script:
```bash
node ./importData.js $AWS_REGION $DDB_TABLE
```

## Test the configuration

**:white_check_mark: Step-by-step Instructions**

1. Confirm that the data is now in the **DynamoDB table** by running the following command:

 ```
aws dynamodb scan --table-name $DDB_TABLE
 ```
This will return all the data in the table together with a "ScannedCount", which is total number of items in the table.

2. Call the **API Gateway Endpoint URL** which SAM has created. First, run the following command in the console to show the endpoint URL:

```
aws cloudformation describe-stacks --stack-name sam-sls-backend --query "Stacks[0].Outputs[?OutputKey=='InitStateApi'].OutputValue" --output text
```
**Note the ```OutputValue``` for the InitStateApi** - this is your API Gateway endpoint. You will need this in later sections.

3. Once you have the endpoint URL, select the URL link in the Cloud9 terminal and select Open: 

![Module 1 open InitStateAPIURL](./assets/images/module2-open-initstateAPIURL.png)

This opens another browser tab and returns all the raw ride and attraction data from the DynamoDB table via API Gateway and Lambda. You have now created a public API that your frontend application can use to populate the map with points of interest.

## Update the Frontend

In this section, you will add the API endpoint you have created to the Frontend configuration. This allows the frontend application to get the list of rides and attractions via the **API-Gateway URL** which pulls the information from **DynamoDB**.

After the update, you will commit the changes to the git repo, which will automatically redeploy and republish the application.

### Update the configuration file

**:white_check_mark: Step-by-step Instructions**

1. In the Cloud9 terminal, in the left directory panel navigate to **sls-frontend/src**. 
2. Locate the **config.js** file and double-click to open in the editor.

This file contains a JSON configuration for the frontend. The file is separated into modules that correspond with the modules in this workshop.

3. In the **MODULE 1** section at the beginning of the file, update the *initStateAPI* attribute of the API by pasting the API Endpoint URL from the previous section between the two ```'```.

4. **Save the file**.

![Module 1 - InitStateAPIURL](./assets/images/module1-initStateAPI.png)

### Push to CodeCommit and deploy via Amplify

1. In the Cloud9 terminal, change to the front-end directory with the following command:
``` 
cd ~/environment/sls-app/sls-frontend/
```
2. Commit to CodeCommit by executing the following commands:
```
git commit -am "Module 1"
git push
```
3. After the commit is completed, go to the [Amplify Console](https://console.aws.amazon.com/amplify/). **Make sure you are in the correct region.**

4. In the *All apps* section, click **sls-frontend**. If you are going back to a previously open browser tab, you may need to refresh.

You will see a new build has automatically started as a result of the new commit in the underlying code repo. This build will take a few minutes. Once complete:

5. Open the published application URL in a browser.

:bulb: The browser may cache an older version of the site - press CTRL+F5 (Windows) or hold down ⌘ Cmd and ⇧ Shift key and then press R (Mac) to perform a hard refresh. This forces the browser to load the latest version.

You can now see the map contains the theme park's points of interest such as rides and attractions. You can select any of them and find out more.

## Module review and next steps

### Module review

In this module you:
- Created a code repository in Cloud9 and configured Amplify Console to publish the web app in this repository. You now have a public URL endpoint for your application.
- Deployed the backend infrastructure for the theme park and application.
- Populated a DynamoDB table containing ride and attraction information for the park. 
- Tested the deployment by using the CLI to scan the DynamoDB table, and using `curl` to test the API Gateway endpoint. 
- Updated the front-end with this new API endpoint and saw the results in the application. 
- Pushed code changes (in the form of a configuration update) to CodeCommit, and saw how Amplify Console automatically detected the new commit and published the changes to the public frontend.

In the next module, you will add realtime waiting times for the rides.

### Next steps

[Module 2. Ride wait times](./README.2.md)
