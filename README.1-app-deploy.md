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