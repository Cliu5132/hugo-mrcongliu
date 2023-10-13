---
title: "Create a Personal AWS Quiz Website"
date: 2023-10-12
---

Last month, I decided to continue my journey on AWS exams. This is idea of acquiring more and more AWS certificates has been in my head for almost a year, since I got my first one called Cloud Practictioner. So I started to watch some courses on Udemy and doing quizes on examtopic.com.

This website is fine overall but it has some annoying features for free users. Well, I don't feel it is necessary to purchase its memebership because I could have built it with ease. So that's what I did: I built my own AWS quiz website. If you are interested in taking a look, try:

- Website: https://aws-saa.mrcongliu.com
- Source Code: https://github.com/cliu5132/data-json

## Overall Serverless Architecture

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/beff492a-9659-ce00-c32a-a2a75c490ce0.png)

- Web Scraping: Developed a custom web scraper to collect AWS certification exam questions and answers from multiple online sources.

- Static Web App: Created an AWS-hosted, static web application using HTML, CSS, and JavaScript for user-friendly access to the collected exam content.

- API Integration: Utilized AWS API Gateway to establish a RESTful API for seamless communication between the frontend and backend components.

- DynamoDB Database: Managed AWS DynamoDB to store and retrieve exam questions, correct answers, and user comments efficiently.

- Serverless Architecture: Implemented serverless AWS Lambda functions for CRUD operations on DynamoDB, optimizing resource usage and cost-effectiveness.

The process can be broken down into the following steps.

## 1. Web Scraping

Well, there are always tools built by others for web scraping. I've been a JavaScript Developer for a while, so I chose Puppeteer JS. It is staightforward, opening a headless browser, clicking a few buttons for you, extracting data from html inner text, and store it in whichever format you like.

```js
const puppeteer = require('puppeteer');
const fs = require('fs');
function delay(time) {
  return new Promise(function(resolve) { 
      setTimeout(resolve, time)
  });
}
(async () => {
  const browser = await puppeteer.launch({
    headless: false, // Set headless to false
  });
  const page = await browser.newPage();
  for (let pageNumber = 17; pageNumber <= 83; pageNumber++) {
    const pageUrl = `https://www.examtopics.com/exams/amazon/aws-certified-solutions-architect-associate-saa-c03/view/${pageNumber}/`;
    // Navigate to the page
    await page.goto(pageUrl);
    //mannual input in case needed
    await delay(9000);
    // Add your logic here to extract data from the current page
    const pageData = await extractDataFromPage(page);
    // Write the accumulated data to a JSON file
    const jsonFileName = `data/exam_data_${pageNumber}.json`;
    fs.writeFileSync(jsonFileName, JSON.stringify(pageData, null, 2));
    console.log(`Data from ${pageData.length} pages has been written to ${jsonFileName}`);
  }
  await browser.close();
})();
async function extractDataFromPage(page) {
  const data = [];
  for (let i = 1; i <= 11; i++) { // Assuming you have 10 questions per page
    const questionXPath = `/html/body/div[2]/div/div[3]/div/div[1]/div[${i}]/div[2]/p[1]`;
    const choicesXPath = [
      `/html/body/div[2]/div/div[3]/div/div[1]/div[${i}]/div[2]/div/ul/li[1]`,
      `/html/body/div[2]/div/div[3]/div/div[1]/div[${i}]/div[2]/div/ul/li[2]`,
      `/html/body/div[2]/div/div[3]/div/div[1]/div[${i}]/div[2]/div/ul/li[3]`,
      `/html/body/div[2]/div/div[3]/div/div[1]/div[${i}]/div[2]/div/ul/li[4]`
    ];
    const question = await page.evaluate((questionXPath) => {
      const element = document.evaluate(questionXPath, document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null).singleNodeValue;
      return element ? element.textContent.trim() : null;
    }, questionXPath);
    const choices = [];
    for (const choiceXPath of choicesXPath) {
      const choice = await page.evaluate((choiceXPath) => {
        const element = document.evaluate(choiceXPath, document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null).singleNodeValue;
        return element ? element.textContent.trim() : null;
      }, choiceXPath);
      try {
        // Remove newline characters and extra spaces
        const cleanedChoice = choice.replace(/\n/g, '').replace(/\s+/g, ' ').trim();
        // Remove "Most Voted" if it exists
        const finalChoice = cleanedChoice.replace('Most Voted', '').trim();
        choices.push(finalChoice);
      }catch(e){
        console.log(choice)
      }
    }
    data.push({ question, choices });
  }
  // Return the extracted data as an object
  return data;
}
```

Our lovely friend Puppeteer will return the data in json format. Oh I love JSON!

```json
[
   {
      "id": 1,
      "single": false,
      "question": "A solutions architect is designing a new service behind Amazon API Gateway. The request patterns for the service will be unpredictable and can change suddenly from 0 requests to over 500 per second. The total size of the data that needs to be persisted in a backend database is currently less than 1 GB with unpredictable future growth Data can be queried using simple key-value requests. Which combination of AWS services would meet these requirements? (Select TWO )",
      "choices": [
         "A. AWS Fargate",
         "B. AWS Lambda",
         "C. Amazon DynamoDB",
         "D. Amazon EC2 Auto Scaling",
         "E. MySQL-compatible Amazon Aurora"
      ],
      "answer": "BC",
      "comment": "In this case AWS Lambda can perform the computation and store the data in an Amazon DynamoDB table. Lambda can scale concurrent executions to meet demand easily and DynamoDB is built for key-value data storage requirements and is also serverless and easily scalable. This is therefore a cost effective solution for unpredictable workloads."
   },
   ...
]
```

## 2. Store Data in AWS DynamoDB

Because the weird data structure and the potential change in the foreseable future, I chose a No-SQL DB to host the data. And AWS DynamoDB is so easy to use, so why not? (Or because I am lazy to switch between different cloud environments, I just want to stay within AWS.)

Currently, we have data in json format, which is create. But to upload it to DynamoDB, we need an uploader. You don't want to do it manually, do you?

Refer to [Amazon DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.client.run-application-python.02-write-data.html)

```python
import boto3


def write_data_to_dax_table(key_count, item_size, dyn_resource=None):
    """
    Writes test data to the demonstration table.

    :param key_count: The number of partition and sort keys to use to populate the
                      table. The total number of items is key_count * key_count.
    :param item_size: The size of non-key data for each test item.
    :param dyn_resource: Either a Boto3 or DAX resource.
    """
    if dyn_resource is None:
        dyn_resource = boto3.resource('dynamodb')

    table = dyn_resource.Table('TryDaxTable')
    some_data = 'X' * item_size

    for partition_key in range(1, key_count + 1):
        for sort_key in range(1, key_count + 1):
            table.put_item(Item={
                'partition_key': partition_key,
                'sort_key': sort_key,
                'some_data': some_data
            })
            print(f"Put item ({partition_key}, {sort_key}) succeeded.")


if __name__ == '__main__':
    write_key_count = 10
    write_item_size = 1000
    print(f"Writing {write_key_count*write_key_count} items to the table. "
          f"Each item is {write_item_size} characters.")
    write_data_to_dax_table(write_key_count, write_item_size)
```

After uploading, the data looks like this:

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/dc0b3402-dc7c-32e3-d7de-be21c8e8230b.png)

## 3. Host Static Web App on AWS S3

S3 is perfect for hosting static websites written in pure HTML, CSS, and JS. I just uploaded all the files there, any enable the website hosting feature.

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/adce5b8f-eb21-caed-0a3f-4c53516d6c0a.png)

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/11da3413-7fee-7041-2f18-47b20ce2418b.png)

## 4. Enable AWS Cloudfront

Cloudfront will cache frequently requested data and provide faster response to users. I must admit this part is a little tricky because you have to give both S3 and Cloudfront a root `index.html` other wise it will appear in your browser url, which is ugly.

Besides, don't forget to give it a SSL certificate, who doesn't like HTTPS?

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/3bd5023c-c2dd-6ccc-fffc-114ecd8dcf55.png)

## 5. Create API Gateway to enable RESTful API

We just need to worry about the front-end, no need to worry about the back-end.

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/d1a67151-302b-4715-70fe-553e1aeb4caa.png)

## 6. Create Lambda functions

To support API Gateway, I created a few Lambda functions to fetch data from DynamoDB and return it the the API routes. Besides, they are very cheap, becuase I will only be charged by the time it is running. I am sure my website won't be used by anyone other than myself. So it will be free 99% of the time.

```js
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import {
  DynamoDBDocumentClient,
  GetCommand,
} from "@aws-sdk/lib-dynamodb";

const client = new DynamoDBClient({});

const dynamo = DynamoDBDocumentClient.from(client);

const tableName = "aws_quiz";

export const handler = async (event, context) => {
  let body;
  let statusCode = 200;
  const headers = {
    "Content-Type": "application/json",
    "Access-Control-Allow-Origin": "*", // Required for CORS support to work
    "Access-Control-Allow-Credentials": true, // Required for cookies, authorization headers with HTTPS
  };

  try {
    // By default, id in url is a string! Convert the id from a string to a number using parseInt
    const id = parseInt(event.pathParameters.id, 10); // Use base 10 for decimal numbers
    
    body = await dynamo.send(
      new GetCommand({
        TableName: tableName,
        Key: {
          id: id, // Now id is a number
        },
      })
    );
    body = body.Item;
  } catch (err) {
    statusCode = 400;
    body = err.message;
  } finally {
    body = JSON.stringify(body);
  }

  return {
    statusCode,
    body,
    headers,
  };
};
```

Don't forget to integrate this lambda function into the API route.

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/95e1cdf3-615e-cf09-4615-a1624cf599a3.png)

## 7. Resolve the subdomain via Route 53

I registered the domain `mrcongliu.com` months ago. While it is hosting my blog. I can assign a sub domain out of it, like aws-saa.mrcongliu.com because it will server the quizes for the aws solution architect associate exam, SAA for short. 

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/7f9cda90-ff98-110f-1a83-2febe30295bc.png)

## Conclusion

It took me a day to build this thing with all of this cool features, and it can make my learning easier without endure the non-free-user-friendly examtopic website. The only way to become a Cloud Practitioner is by building cool staff using Cloud Tech. 