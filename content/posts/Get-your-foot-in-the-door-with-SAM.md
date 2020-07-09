---
title: "How to get your SAM🐿 Hello World done easily"
date: 2020-07-08T14:37:53+02:00
draft: false
---

## - How to get your SAM Hello World done easily -
_A much easier to follow tutorial than AWS' own._

![SAM_Picture](https://i.ytimg.com/vi/1dzihtC5LJ0/maxresdefault.jpg)

If you are interested in SAM, you probably already skimmed through the [Getting Started with SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-getting-started-hello-world.html) page and decided it's way too complicated for you. Well fear none! We'll do something much more linear here instead of that "all-in-one" deployment.

### A little backstory
After having deployed a complete serverless architecture with pure CloudFormation, Forrest Brazeal suggested I look into the SAM framework. I was reluctant at first, thinking _"how could 20 lines of code replace my gorgeous 450 line template!?"_ - but he was right. Although it didn't entirely eradicate the need for some CF resources, the ones that are supported by SAM have seen a code shrinking of ~70%! This is not the only great part about it however, the deployments are much more informative, interactive and _easy_! While CloudFormation gives you a simple _"hey, this is the update ID, do what you want with it"_, SAM gives you a detailed walktrough of what it will do and asks you if you are **sure** you want to deploy those changes. Finally, it saves your configuration in a file so you don't have to mess around with command line flags next time.

## Prerequisites
To begin I will assume that you have the following already set up:
- Configured AWS credentials in your CLI
- The IAM user of those credentials has access to S3, CloudFormation, CodeDeploy, Lambda, IAM & API Gateway (or `AdminAccess`)
- 🐳 Docker [installed](https://docs.docker.com/desktop/#download-and-install) and running (only for the more advanced section)
- Your favourite snack in your _left pocket_ (this is very important! will come back to this later)


⏱ _Project length: ~45-60 minutes_ - 💸 _Costs: Included in the free tier, otherwise less than $0.00001_

## Getting Started
_This post's code is published on my [GitHub](https://github.com/what-name/blog-tutorial-assets/tree/master/start-with-sam-01), however I would highly suggest that you follow along manually instead of just cloning it from there - you are here to learn._

### Install the SAM CLI
If you are on Windows, [here's the official guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install-windows.html#serverless-sam-cli-install-windows-sam-cli).
If you are on MacOS or Linux, installing the CLI is super easy with `brew`.
{{< highlight "linenos=table" >}}
brew tap aws/tap
brew install aws-sam-cli
{{< /highlight >}}

SAM uses the same credentials as your regular AWS CLI, so there is no extra configruation needed. Open up your favourite editor (VSCode for example) with the folder you want to store your SAM project in, and create a file called `template.yaml`.[1] Put the following in there:
{{< highlight "linenos=table" >}}
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: >
  My SAM App

  A much easier to follow tutorial than AWS' own.

Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
{{< /highlight >}}

That is it for now! As I said, we are not doing a full-fledged deployment out of the gate, that is very confusing (as it was for me). Firstly we will get comfortable with how SAM works, then let it flex its muscles later. Luckily, this is enough to get started because **SAM supports pure CloudFormation resources.** This means that SAM can build & deploy a pure CF template without any sam-specific resources! You can't build the next Billion dollar app right away, but you can deploy an S3 bucket today and increment as you go!😉

Next up, we need to _**build**_ the project and then _**deploy**_ it. Open up a terminal in your project's folder and execute the following commands:
{{< highlight "linenos=table" >}}
sam build
sam deploy -g
{{< /highlight >}}

The `-g` flag means `guided`. It will nicely ask you about some extra things it needs:
{{< highlight "linenos=table" >}}
Looking for samconfig.toml :  Not found

        Setting default arguments for 'sam deploy'
        =========================================
        Stack Name [sam-app]: my-sam-app
        AWS Region [us-east-1]: us-east-1
        #Shows you resources changes to be deployed and require a 'Y' to initiate deploy
        Confirm changes before deploy [y/N]: Y
        #SAM needs permission to be able to create roles to connect to the resources in your template
        Allow SAM CLI IAM role creation [Y/n]: Y
        Save arguments to samconfig.toml [Y/n]: Y

        Looking for resources needed for deployment: Not found.
        Creating the required resources...
{{< /highlight >}}

Lets go through this in detail:
- **Stack Name**: The name of the stack that will be deployed. Self-explanatory but there will be some extra random characters added at the end automatically to avoid name conflicts.
- **AWS Region**: The region where the stack will be deployed.
- **Confirm changes before deploy**: This will prompt you again before deploying the changes. See screenshot below.
- **Allow IAM role creation**: SAM's power is in its simplicity. This means that we are not required to create seperate IAM resources for Lambda functions for example, it is all taken care of. No need for an IAM resource with S3, but we'll just say `Y` to save it for future deployments.
- **Save arguments to `samconfig.toml`**: This is important. Remember when I said it saves the settings into a config file? This is it. After the deployment has succeeded, check the contents of that file and see what's inside! If that file is configured, you can omit the `-g` flag as it will read everything from there. You are free to modify the contents in there as well - but very cautiously, you don't want to break anything.

After a bit of thinking, it returns with a nice list of resources that will be added, modified or deleted. This is where you can double check if everything looks good and that only the resources will be touched that need to be. If all looks good, `Y`. Watch your terminal as the job gets done and you should be treated with a `Successfully created/updated stack` message. _Oh that sweet feeling!_

![sam_01](/images/sam-easy-tutorial/sam_01.png)

If you go onto your AWS Console into S3, you will see two buckets there. One is called `aws-sam-cli-managed-default...`, this is for SAM to store its files and configurations for all your projects. The other one is `my-sam-app-s3bucket-randomchars`, which is the bucket we defined in the `template.yaml` file under "resources". So far so good!

### ⏳ Pause
What did we just do? We installed the SAM CLI, created a template with a single resource (an S3 bucket), built the template with `sam build` and deployed it onto our AWS account with `sam deploy`. This is the base knowledge needed to use SAM and this is one main things that seperates it from pure CloudFormation. Where instead of `aws cloudformation create/update-stack`, SAM uses a more interactive deployment strategy that lets you know _what will happen exactly_.

### ▶️ Advancing further
Now that we have a firm grasp on the basics, lets create a very simple Lambda function with an API Gateway that will return a "Hello World" to an HTTP request. Crate a folder in your project's directory called `Lambda` and a file inside there called `app.py`. Our function at this point will only return a very simple text, no need for anything fancy yet. The `app.py` file should have this in it:
{{< highlight "linenos=table" >}}
import json

def lambda_handler(event, context):
  return {
      "statusCode": 200,
      "body": json.dumps({
          "message": "SAM is great!",
      })
  }
{{< /highlight >}}

Then we need to add the Lambda function to the template. Pay close attention to the indentation, as yaml is very picky about that (rightfully so). This is what the `Resources` section should look like:
{{< highlight "linenos=table" >}}
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

🤠 _"Wow, okay, slow down cowboy, what does this mean?"_ Lets break it down:
- **HelloWorldFunction**: Simply the name of the resource, can be anything but keep it informative and unique throughout the template
- **Type**: The type of the resource, in our case a `Function`
- **CodeUri**: The folder inside the project where our Lambda function lives
- **Handler**: filename.functionname referencing the app.py file and its function
- **Runtime**: The runtime environment. Can be Python, NodeJS, Java, etc.
- **Events**: The events that will trigger our function
  - **HelloWorld**: The name of the event, can be anything unique
    - **Type**: Here this means an API Gateway resource
    - **Properties**: Properties of the API Gateway
      - **Path**: The endpoint of the API Gateway, can be anything. This will be in the form of `https://apigw_url.com/Prod/hello`
      - **Method**: What HTTP method will the gateway respond to (get, post, etc.)
- **Outputs**: This will be printed to the console when deployment finishes. It will give us back the API Gateway's endpoint URL where we can interact with the new Lambda function.

Save both of the files and create an **empty** file called `requirements.txt` inside the `Lambda` folder. This file is required and it is where all the extra libraries need to be detailed that the function will use - for us, nothing. This is what our directory structure should look like:
{{< highlight "linenos=table" >}}
start-with-sam
|____template.yaml
|____samconfig.toml
|____Lambda
| |____app.py
| |____requirements.txt
|____.aws-sam
| |____build
| | |____template.yaml
{{< /highlight >}}

#### Lets build again!
We now have a Lambda function ready to go. Keep in mind that at every change, the SAM project needs to be rebuilt. This time however, we will use the `sam build --use-container` command. The `--use-container` flag will instruct SAM to use a Docker container for building our project. This is a great feature because it lets us keep our local development environment clean and it automatically fetches a container with everything already preinstalled . Without this flag, we would have to make sure that we have the runtime in our `PATH`, all the necessary bells and whistles installed and it's just generally a more painful experience. Why not use something that is purpose-made if it's just a flag away! Before you run this however, make sure that 🐳Docker is running. If all went well, you should be presented again with that sweet `Build Succeeded` output.

#### Lets deploy again!
SAM is so easy to use after we get the hang of it. We just created a brand new Lambda function, an API Gateway and all the necessary IAM roles with only 12 lines of code! To deploy, we don't need to do anything unusual, just a simple `sam deploy`. It will again ask us to confirm the `changeSet` (you can turn this off in the `config.toml` file) and if everything looks good, simply input `Y`.

![sam_02](/images/sam-easy-tutorial/sam_02.png)

Now sit back, wait for it to finish and click the URL in the output. You should see a beautiful response coming back to you in your browser from the Lambda function. This means that your new SAM application was sucessfully deployed!🥳

![sam_03](/images/sam-easy-tutorial/sam_03.png)

### Destroy it all!😈
If you go into your AWS Console, you can see all the resources that were deployed in this project. The Lambda function, some IAM roles, the S3 bucket and of course the CloudFormation stack. **Remember, SAM uses CloudFormation in the background, it translates every SAM resource into pure CF and deploys it as a stack.** To delete everything, simply go to the CloudFormation console, choose the `my-sam-app` stack and hit "delete". The other stack `aws-sam-cli-managed-default` is a bit more difficult to delete, but you don't actually have to. This is the stack that controls SAM's S3 bucket where the projects' files live. If you deploy a new (or different) application in the future, you can simply copy the `s3_bucket` value into the new application's `config.toml` file and SAM will use that bucket for multiple applications. It will not cost you anything if you leave it like that. It's quite difficult to destroy that bucket because it is a **versioned** one, therefore all the previous versions of all objects have to be deleted with Lifecycle Policies before the bucket itself can be deleted.

## Closing thoughts
**🍩 Remember that snack in your left pocket? Take it out and enjoy, you deserve every bit of it!**

I hope you enjoyed this tutorial [🥚 _at least half as much as we enjoyed playing it for you_] and gotten a bit more familiar with SAM🐿. If you have any questions, thoughts or problems you need help with, you can hit me up on [Twitter](https://twitter.com/chris_the_nagy)!

In the next part of this, we will add a DynamoDB table into the mix, change our Lambda function a bit and create a simple incremental counter that works with HTTP requests through your browser!


* * *
#### Footnotes
[1] The template file can be called anything, `template.yaml` is the default one, but if you want to use a custom name you can simply use the `-t my-template-name.yml` flag when building and deploying.

_🗓 This post was published on the 8th of July, 2020._