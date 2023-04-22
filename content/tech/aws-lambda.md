---
title: "An Introduction to AWS Lambda"
date: 2023-04-21
---
# What is AWS Lambda

AWS Lambda is a revolutionary new way to build and run applications without the need for dedicated servers or infrastructure. With Lambda, you can simply upload your code, and AWS takes care of everything else.

At its core, Lambda is a computing service that lets you run your code in response to events and automatically manages the compute resources for you. This means you don't need to worry about setting up servers, scaling, or even patching operating systems. AWS handles all of that for you.

For example, I want to build a service that automatically runs every 9:00 am each day. Using a server probably is not cost-effective for such a simple task. Instead of renting a real server, Lambda is a great choice in this use case cause I only pay for the time I actually use it, which might be less than 1~2 seconds each day.

But for now, let's try to start with something simple. Let's build a serverless function that will say "Hello world!" when it is triggered.

# How to use AWS Lambda

Here's a step-by-step guide on how to build a Lambda function using Node.js and the Serverless Framework and deploy it to AWS:

### Step 1: Install Serverless Framework

To get started, you'll need to install the Serverless Framework. Run the following command to install it globally:

```bash
npm install -g serverless
```

### Step 2: Create a new Serverless service

Create a new directory for your Serverless service and navigate to it. Then, run the following command to create a new Serverless service:

```bash
serverless create --template aws-nodejs --path mrcongliu
```

This will create a new directory called `mrcongliu` with some boilerplate code for a Serverless service using Node.js.

### Step 3: Write your Lambda function

Open the **`handler.js`** file in the `mrcongliu` directory and write your Lambda function code in it. Here's an example that simply returns a string:

```jsx
module.exports.hello = async (event, context) => {
  return {
    statusCode: 200,
    body: JSON.stringify({ message: "Hello, world!" })
  };
};
```

### Step 4: Configure Serverless service

Open the `serverless.yml` file in the `mrcongliu` directory and configure your Serverless service. Here's an example configuration that sets the function name, runtime, and handler:

```yaml
service: mrcongliu

provider:
  name: aws
  runtime: nodejs14.x

functions:
  hello:
    handler: handler.hello
```

### Step 5: Test locally

There are two ways to test a Lambda function, either passing in a data object to it or sending a event.json to it. 

#### First Approach - Passing A Data Object

If your data is small in size or you just don't want to get too complicated into the configuration, use a data object. The syntax of passing a `—data {object}`looks like below.
     
```bash
serverless invoke local -f hello --data '{\"name\":\"mrcongliu\"}'
```

The reason you have to add those backslashes is because it will throw an error if you don't.

```bash
serverless invoke local -f hello --data '{"name":"mrcongliu"}'
```
#### Second Approach - Passing An EVENT.JSON
I would prefer this way cause I don't need care about my input format. I simplely put everything inside `event.json`.

```jsx
serverless invoke local -f hello -p event.json
```

### Step 6: Deploy to dev

Run `sls deploy` from the root of a service which you want to deploy.

### Step 7: Test on the dev

Three ways to test a deployed AWS Lambda function: the previous two ways plus dashboard triggering.

#### 1. (Same as 1st for local) —data {object} 
    
```bash
serverless invoke -f hello -s dev -l --data '{\"name\":\"mrcongliu\"}'
```
{{< notice tip >}}
Note there is one major difference we need to pay attention to, which is the logs will be missing from our terminal. Because the logs (on the cloud) by default will not be returned from AWS. Though we can check the logs on AWS CloudWatch later, but I prefer seeing the logs in real time. Therefore, I use this `-l` command to enable real-time logs in my local terminal.
{{< /notice >}}

In the serverless invoke command, the -l option stands for "logs". When this option is used, it tells the Serverless Framework to stream the logs of the invoked function to the console, so that you can see the output of the function as it runs.

Here's what the other options in the command do:

- `-f hello`: specifies the name of the function to invoke. In this case, the function is called "hello".

- `-s dev`: specifies the name of the stage to invoke the function in. In this case, the stage is called "dev".

- `-p event.json`: specifies the path to a JSON file containing the input event data for the function. The event data is passed to the function as an argument when it is invoked.

#### 2. (Same as 2nd for local) —path event.json

```bash
serverless invoke -f hello -s dev -l -p event.json
```

#### 3. (Only for deployed functions) Dashboard Triggering

1. Log in to the AWS Management Console and navigate to the Lambda service.
2. Click on the name of the function you want to invoke.
3. In the "Function overview" section, click on the "Test" button.
4. In the "Configure test event" dialog box, select the "Hello World" template or create a custom test event with the desired input.
5. Click on the "Create" button to create the test event.
6. Click on the "Test" button to invoke the function with the test event you just created.
7. The result of the function invocation will be displayed in the "Execution result" section at the bottom of the page.
    
{{< notice note >}}
Note that all of these methods are primarily intended for testing purposes and should not be used for production-level invocations. 

For production-level invocations, you should use an API Gateway or another AWS service to trigger your Lambda function.
{{< /notice >}}