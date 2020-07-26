---
title: "Use DynamoDB with SAM🐿 - SAM_02"
date: 2020-07-24T15:52:11+02:00
draft: false
---

# Add DynamoDB to a SAM Application and connect it to a Lambda function - SAM_02

[In my last post about SAM](https://blog.heyitschris.com/posts/get-your-foot-in-the-door-with-sam/), I talked about how to get started with the very basics. It got such a warm response from the community that I decided to write more about SAM. (and also because I just love it😍) Lets not waste more time and get into it!

## What we'll add ⚙️
If you haven't done [the previous tutorial](https://blog.heyitschris.com/posts/get-your-foot-in-the-door-with-sam/), you can go through that first or [clone the repository from my GitHub](https://github.com/what-name/blog-tutorial-assets/tree/master/start-with-sam-01)(`start-with-sam-01`). Here's what we'll add in this one:
- A DynamoDB table
- A Lambda function that will read & write an item from the table
- An API Gateway that will connect us with DynamoDB

This is a great use case for a SAM application - building APIs. Serverless applications are by design highly scalable and managed by the provider, this lets us focus on all the other parts and not have to worry about keeping our service running - AWS handles that🙂 (or rather the smart folks who work there😉)

## Modifying the template 🛠
This is our `template.yaml` currently:
{{< highlight "linenos=table" >}}
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: >
  My SAM App
  A much easier to follow tutorial than AWS' own.
Resources:
  S3Bucket:
    Type: AWS::S3::Bucket

  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: Lambda/
      Handler: app.lambda_handler
      Runtime: python3.8
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get

Outputs:
  HelloWorldApi:
    Description: "API Gateway endpoint URL for Prod stage for Hello World function"
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/hello/"
{{< /highlight >}}

### Add a DynamoDB table 📇
Luckily this doesn't need much modification. You can remove the S3 bucket if you want, but I always like to just have a random S3 bucket in there, you never know😉 What we'll add however is a `DynamoDB` resource, or rather a `SimpleTable`. As I said before, SAM can deploy and use _pure_ CloudFormation resources as well as SAM resources. We have the option here to either go with a `AWS::DynamoDB::Table` or a `AWS::Serverless::SimpleTable`. Both will result in the same DynamoDB table, but the latter needs much less configuration while the first is much more configurable. We'll go with SAM's own. Here's the template reference for [SimpleTable](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-resource-simpletable.html). Let's add the following just above the `Outputs` line (while paying attention to indentation!):

{{< highlight "linenos=table" >}}
MyTable:
  Type: AWS::Serverless::SimpleTable
{{< /highlight >}}

**WAIT, that's it?🤯** Yeah, that's literally all you need to create a DynamoDB table with SAM. There are more properties that you can define however as you can see from the above mentioned template reference, so lets go through them:
![sam-02-1](/images/sam-02/sam-02-1.png)
- **PrimaryKey:** This is your table's Primary Key (PK). Every single table in DynamoDB has to have a PK. This is the unique identifier for every single item (we can use _secondary keys_ as well but that's for later). If not defined, the PK of the table will be `id` by default. Here's how you can define your Primary Key manually:
{{< highlight "linenos=table" >}}
MyTable:
  Type: AWS::Serverless::SimpleTable
  Properties:
    PrimaryKey:
      Name: myKeyName
      Type: String
{{< /highlight >}}

- **ProvisionedThroughput:** This property sets if you want to use provisioned throughput, or use the `PAY_PER_REQUEST` model. Pay per request means that you don't have to worry about absolutely anything when it comes to performance. I won't get into much detail on how DynamoDB handles reads and writes, but this model is amazing to have if you don't know yet how much performance you'll need out of your table. If you don't define the provisioned throughput, the `PAY_PER_REQUEST` model will be used by default and you don't need to define that seperately.
{{< highlight "linenos=table" >}}
MyTable:
  Type: AWS::Serverless::SimpleTable
  Properties:
    PrimaryKey:
      Name: myKeyName
      Type: String
    ProvisionedThroughput:
      ReadCapacityUnits: 5
      WriteCapacityUnits: 5
{{< /highlight >}}

- **SSESpecification:** Server Side Encryption. This is what you need to define if you want server side encryption enabled on the table. This is something that's beyond the scope of this project so I won't go into detail, you can find more details about it [here](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-dynamodb-table-ssespecification.html).
    
- **TableName:** Self explanatory. The great thing about SAM and CloudFormation is that you do not need to explicitly define a name for any resource. In the very first example, we didn't define one either. What happens in this case is SAM takes the name of the application defined in the `samconfig.toml` and uses that as the root of the name. Then it will use the name of the resource in the template, so in our case `MyTable`. After this it will add a string of random characters in order to avoid any name conflicts with other resources. The final name of the table (and any resource for that matter) will be something like: `MySamApplication-MyTable-hf8kac2`. It is very much encouraged **not to name your resources** because that can cause unforseen issues. They are rare but just trust me (and the people who I've learned this from) on this one.

- The last one is tags, which are self explanatory: `TagName:TagValue`.

- Here's what a fully custom `SimpleTable` resource would look like: (_**you do not need to use this, it's just FYI**_)

{{< highlight "linenos=table" >}}
MyTable:
  Type: AWS::Serverless::SimpleTable
  Properties:
    TableName: uEvenTableBro
    PrimaryKey:
      Name: id
      Type: String
    ProvisionedThroughput:
      ReadCapacityUnits: 5
      WriteCapacityUnits: 5
    SSESpecification:
      SSEEnabled: false
    Tags:
      CostCenter: tutorials
      ChrisLovesSAM: true
{{< /highlight >}}

### Deploy! 🛫
Lets deploy the template! It's always a great idea to deploy things in pieces in order to isolate issues. If you have a typo or small issue in your table resource, you will see it here. If you don't deploy now and have an error in another component later down the road, it will be much more difficult to troubleshoot all the changes that you've made at once.

Remember that whenever you make changes to a SAM template or any of the files in the project, you always need to do `sam build` and I like to use the `--use-container` flag as explained before. So the full command is `sam build --use-container && sam deploy` (make sure that your `samconfig.toml` is up to date, or use the extra `-g` flag for a guided deployment).

## Lambda 🔑
_(Please don't hate me Lambda pros, I know my Python🐍 isn't perfect😇)_
This is where we'll modify our original Lambda function to communicate with the DynamoDB table. Currently all it does is return a hardcoded string to us. This isn't really useful anymore so let's open up the `Lambda/app.py` file and replace everything in there with this:
{{< highlight "linenos=table" >}}
import json
import boto3
import os

# Initialize dynamodb boto3 object
dynamodb = boto3.resource('dynamodb')
# Set dynamodb table name variable from env
ddbTableName = os.environ['databaseName']
table = dynamodb.Table(ddbTableName)

def lambda_handler(event, context):
    # Update item in table or add if doesn't exist
    ddbResponse = table.update_item(
        Key={
            'id': 'HelloThere'
        },
        UpdateExpression='SET generalle = :value',
        ExpressionAttributeValues={
            ':value': 'Hello there, general Kenobi.'
        },
        ReturnValues="UPDATED_NEW"
    )

    # Format dynamodb response into variable
    responseBody = json.dumps({"HelloThere": ddbResponse["Attributes"]["generalle"]})

    # Create api response object
    apiResponse = {
        "isBase64Encoded": False,
        "statusCode": 200,
        "body": responseBody
    }

    # Return api response object
    return apiResponse
{{< /highlight >}}

**I DON'T UNDERSTAND A SINGLE THING IN THERE!🥵** Don't worry! It's okay if you don't, just simply copy-paste it into your `Lambda/app.py` file! I don't really want to get into every line of that code because we could be here until tomorrow, but here's what's important:
- **ddbTableName = os.environ['databaseName']** - our Lambda function needs to know which DynamoDB table we want it to interact with. We could easily hardcode the table's name into the function, but that is very, _very_, **very** bad practice. We'll use _environment variables_ which I'll talk about down below as well.
- **ddbResponse = table.update_item()** - this is the command that will _update_ the item in our database table and return us the value. Strictly speaking this is an _update_ operation, but in DynamoDB this can be used to create an item if it doesn't exist yet, which will happen in our case as well. Usually you'd use the `get_item()` operation to simply get an item from a table, but again, we don't have an item yet so this will just create it for us on the fly.

### Environment variables for Lambda
As I just mentioned, we'll use _env variables_ inside the Lambda function to reference our table. We need to pass this value from the SAM template itself! The way to do this is a simple `Environment:` property inside our Lambda resource. The LambdaFunction resource inside the `template.yaml` should look like this:
{{< highlight "linenos=table" >}}
StartWithSamFunction02:
  Type: AWS::Serverless::Function
  Properties:
    CodeUri: Lambda/
    Handler: app.lambda_handler
    Runtime: python3.8
    Events:
      HelloWorld:
        Type: Api
        Properties:
          Path: /hello
          Method: get
    ### LOOK UNDERNEATH HERE!!!!!
    Environment:
      Variables: 
        databaseName : !Ref MyTable
{{< /highlight >}}

As you can see, there's a new property there called `Environment`. This is where we can define environment variables that will be passed along to the Lambda function. There's a new concept here I think, the `!Ref` command. In a nutshell, every resource in CloudFormation has return values such as the name of the deployed resource or its _ARN_. The `!Ref` command in our case will return the _name_ of the DynamoDB SimpleTable. Make sure that the `MyTable` value is the exact same as the *name* of your SimpleTable resource (as you can see above). Since we never give explicit names to our resources as I said before, this is the one of the ways how we pass values from one service to another. The actual value of the `!Ref MyTable` will look something like this: `my-sam-app-02-myTable-niv234fs2`, but we don't really care about that honestly, it's all taken care of by these variables and SAM. **Note that the `databaseName` value will be the name of the environment variable that gets exposed to Lambda! It has to be the same such as `os.environ["databaseName"]`.**

**❌Don't deploy yet!❌**

### Lambda's permission to DynamoDB 🤔
If you'd deploy this right now, the Lambda function would not be allowed to access anything in any database, or any resource in the whole universe for that matter. Here another SAM capability comes in handy, _[policy templates](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-policy-templates.html)_. A policy template is a pre-made (by the SAM folks) IAM policy that we can attach to any of our resources. These are great because instead of manually creating roles, we can just attach an existing policy that'll allow us everything we need. This is what you need to add under the `Environment` with the **same indentation**!

{{< highlight "linenos=table" >}}
Policies:
  - DynamoDBCrudPolicy:
      TableName: !Ref MyTable
{{< /highlight >}}

And this is what you should get as a **final** `template.yml` file:

{{< highlight "linenos=table" >}}
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: >
  My SAM App 02

Resources:
  StartWithSamFunction02:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: Lambda/
      Handler: app.lambda_handler
      Runtime: python3.8
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get
      Environment:
        Variables: 
          databaseName : !Ref MyTable
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref MyTable

  MyTable:
    Type: AWS::Serverless::SimpleTable
    
Outputs:
  HelloWorldApi:
    Description: "API Gateway endpoint URL for Prod stage for Hello World function"
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/hello/"
{{< /highlight >}}

### Where we at
Right now we created the DynamoDB table, modified the Lambda resource in the template to use environment variables (for the DynamoDB table) and to use a policy template to access said table. We also modified our Lambda's code to interact with the database (although in a very simple way).


### Deploy 🚀
The usual `sam build --use-container && sam deploy (-g)`. As always, if there are any errors, use the `sam deploy --use-container --debug` command to see what's happening. 🚨 _A quick note here, if you are on the SAM CLI Version1.0, you might encounter problems when using the `--use-container` flag. Try downgrading to Version0.53 and that will solve it._

### Does it even work? 🥵
In your terminal, you should get the output of the API URL as seen below. Click that and check if you are getting something returned that doesn't look like an error. If that works, go to your DynamoDB table and see if the new item is indeed there! You can also modify the item's `generalle` attribute to something else, request the API URL again and see if it changed it back😉 (FYI, it should)

![sam-02-2](/images/sam-02/sam-02-2.png)

## What did we learn 🤓
- How to create a DynamoDB table with SAM
- Use Lambda to interact with a DynamoDB table
- How to use **environment variables** with Lambda in a SAM template
- How to use SAM **policy templates**

## Closing words 🥳
As always, if you have any issues or thoughts, feel free to hit me up on [Twitter](https://twitter.com/chris_the_nagy)! This project's code can be downloaded from the [GitHub repository](https://github.com/what-name/blog-tutorial-assets) as well, under `start-with-sam-02`.

In the _next episode_, I'm not sure what I'll talk about next, but it's gonna be fun😜

### Random resources you should check out
- [Sessions with SAM playlist](https://www.youtube.com/watch?v=AdOoQKsrBQw&list=PLJo-rJlep0ED198FJnTzhIB5Aut_1vDAd) - this is an amazing playlist by (mostly) [Eric Johnson](https://twitter.com/edjgeek) on some advanced SAM magic! _I'm in dire need of that squirrel onesie btw🐿_
- [DynamoDB boto3 cheat sheet](https://dynobase.dev/dynamodb-python-with-boto3/) - how to interact with DynamoDB from boto3 basics
- [AWS SAM Resource and Property Reference](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification-resources-and-properties.html)

_🗓 This post was published on the 26th of July, 2020._