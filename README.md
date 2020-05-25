# Serverless Application Workshop

You will be developing & deploying a completely Serverless Web Application built with AWS services.

## Welcome to the Theme Park!

This exciting new **Theme Park**, combines roller coasters and rides with shows and exhibits. The park will open every day and expects up to **50,000 visitors daily**. It's self-guided, using a Web Application that Guests can browse on their Smartphones. 

The only "slight" problem is that the development team has suddenly left and the park's Grand Opening is today! You only have 4 hours to finish assembling the remaining pieces of the application before the gates open. But don't worry, **Serverless** is at hand! These instructions will guide you through using AWS Services to assemble a complete application so you can save the day.

## Application structure

You will be using a Micro-Services approach to configure a Frontend and build a Backend Serverless Application.

### Frontend

The Frontend Web Application consists of an existing JavaScript Web Application managed with [AWS Amplify Console][amplify-console] that interfaces with Services on the Backend. You will only need to make minor changes to a configuration file in the Frontend code to complete this workshop.

Amplify Console provides a simple, Git-based workflow for deploying and hosting Fullstack Serverless Web Applications. Amplify Console can create both the Frontend and Backend but for this workshop we will be using Amplify Console for only the frontend.

Amplify will be used to host static web resources including HTML, CSS, JavaScript, and image files which are loaded in the user's browser via S3. 

### Backend

The Backend Application Architecture uses [AWS Lambda][lambda], [Amazon API Gateway][api-gw], [Amazon S3][s3], [Amazon DynamoDB][dynamodb], and [Amazon Cognito][cognito]. 

JavaScript executed in the Frontend browser application sends and receives data from a public Backend API built using API-Gateway and Lambda. DynamoDB provides a persistence data storage layer which is used by the API's Lambda functions.

See the diagram below for the complete architecture.

![Overall architecture](./assets/images/architecture.png)

## Start here

After setting up your **Cloud9** environment, follow the modules in order:

Module # | Feature | Description
------------ | ------------- | -------------
1 | Deploy the App | Deploy the initial frontend and backend applications.
2 | Ride wait times | Integrate your application with the ride systems so guests can see wait times.
3 | Ride photos | Build a photo processing flow so guests can take selfies around the park.
4 | Translation | Help international guests understand the app by adding language translation.
5 | Analyzing visitor stats | Collecting and analyzing large amounts of data from park guests.
6 | Developing event-based architecture | Routing park maintenance events depending upon severity.

If you run out of time in the workshop, don't panic! This GitHub repository is public and is available after your workshop ends.

## Deleting resources

If you are using an account provided at an AWS event, the account will be cleaned up automatically. 

If you are using your own AWS account, this workshop uses AWS services that are mostly covered by the Free Tier allowance (if your account is less than 12 months old) but it may incur some costs. To minimize cost, make sure you deprovision and delete those resources when you are finished.

**:loudspeaker: You are liable for the costs incurred of running this workshop.**

### Next steps

:white_check_mark: Proceed to the [Module 1](./README.1-app-deploy.md), where you'll start setting up your application.

[amplify-console]: https://aws.amazon.com/amplify/console/
[cognito]: https://aws.amazon.com/cognito/
[lambda]: https://aws.amazon.com/lambda/
[api-gw]: https://aws.amazon.com/api-gateway/
[s3]: https://aws.amazon.com/s3/
[dynamodb]: https://aws.amazon.com/dynamodb/
