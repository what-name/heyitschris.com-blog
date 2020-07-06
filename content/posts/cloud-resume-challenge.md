---
title: "The Cloud Resume Challenge"
date: 2020-06-23T19:23:25+02:00
draft: false
---

## The Cloud Resume Challenge was way more fun than I thought

A.K.A. the story of [heyitschris.com](https://heyitschris.com)

### ⚠️Update
- This was initially published on Medium. Since I created my own blog, I saw no reason not to migrate this post to this platform.
- Since this post was published, I have fixed all the small issues on the infrastructure side, improved the CI/CD workflow, implemented unit tests for my Lambda function and converted the CloudFormation template to SAM. [See my GitHub for the project's repository](https://github.com/what-name/heyitschris.com-backend).

One afternoon I came across a Reddit post; “The Cloud Resume Challenge by Forrest Brazeal” and it peaked my interest. I was just starting to study for my AWS Solutions Architect Associate certification at that time, and I dived right into it. I love projects, projects are the primary way I learn and I knew this challenge was going to solidify most parts of what I have been studying for the exam.

{{< amp-iframe
    sandbox="allow-scripts allow-same-origin"
    src="https://giphy.com/embed/eHFSpEGnvxSI8NpTaS"
    width="480"
    height="321"
    layout="responsive" >}}

## The initial deployment

I went right to work and had a functioning website by the end of the day initially. It was a very rough **MVP** but it allowed me to visualize and realize the outcome of the project. For me, the more I visualize something, the longer I can stick to it without dragging it even a tiny bit. In my mind, the project was already finished, time and “work done” just haven’t caught up yet.

{{< amp-iframe
    sandbox="allow-scripts allow-same-origin"
    src="https://giphy.com/embed/26xBMwu2cs1ClnGWQ"
    width="480"
    height="321"
    layout="responsive" >}}

## [The Certificate](https://www.youracclaim.com/badges/60615e70-6409-4752-9d77-3553a43d13d2/public_url)

The #1 point requires a cloud certification from the provider of your choice. Since I was already on my way to one from AWS, this was an obivous one: I needed to pass the cert. I originally studied on [CBTNuggets](https://cbtnuggets.com/) and I was very satisfied with Bart Castle’s course. Of course I knew that this isn’t going to be enough, so I made sure to dive deep into the AWS Whitepapers, the FAQs of all main services as well as create small projects on my AWS Account and finish a couple of AWS’ QuickLabs. While the current COVID crisis was going on, this was the perfect opportunity as time was more or less in abundance. After approx. 4 months, I signed up for my test in person and passed it! **#1 Done!**


## The Frontend

The MVP was a very basic site with a lot of lorem ipsum. I do have some minor HTML/CSS experience but knew that this was not the center of this challenge. It is surprisingly difficult to find a good-looking and free resume website template (specific much?), but DuckDuckGo has blessed me and I was presented with the [Beckham template by Colorlib](https://colorlib.com/wp/template/beckham/). I still needed to make numerous changes and even fixes to the template, but all-in-all the frontend part took no more than 12 hours in total. **#2/3 Done!**

![heyitschris screenshot](/images/1*gS2h9LjPNn3vAOLlkDgWMw.png)

## The Backend

From previous small projects along my learning path, I already knew how to deploy an S3 static site, restrict access to an OAI and deploy a CloudFront distribution in front of if, while also paying attention to security. The vision was therefore clear, but I could not accept using the console and manually building things. As per the latest best practices in CI/CD, I opted to create multiple cloudformation templates and create a deployment pipeline for both the backend infrastructure and the frontend code. The decision was made to go with CloudFormation instead of Terraform or the AWS CDK. The first I already have some experience in and I wanted this to be pure AWS. The second requires further knowledge of [for me] Python which I am currently taking courses on.

{{< amp-iframe
    sandbox="allow-scripts allow-same-origin"
    src="https://giphy.com/embed/WmdLOpnd6sBcnkXgSD"
    width="480"
    height="321"
    layout="responsive" >}}


The CI/CD pipeline is powered by GitHub Actions. I hesitated between that and using CodePipeline, but I figured both require similar buildspec files, and I should dive deeper into GitHub a bit as well. Setting up the two Actions was not too difficult, there are great resources on the topic online and also multiple official GitHub guides. One very important detail to pay attention to is never to commit access keys into your code, **ever!** Simply removing a file after it has been merged to the repo does not help, that’s the whole point of source control, it will be there forever. With that in mind, GitHub Secrets came to the rescue where the access keys of the `github-ci/cd-user` are stored safely and referenced in both deployment jobs. **#14/15 Done!**

### 🚨 Never commit your access or secret keys into your repository, ever!

AWS has near impeccable documentation on almost all of their tools - what they do and how - but there were instances where I had to bang my head against the wall for a while. This includes for example CORS errors for the API Gateway service, and to deploy the visitorCount item automatically in the DynamoDB database on initial deploy (something which still hasn’t been implemented yet by me). While working on this project I have focused strongly on making this template as portable as possible. So in the future I can reference it or if needed or somebody can copy out parts of it from my GitHub repository. **#12/13 Done!**

## The Visitor Counter

This was an interesting challenge as it encompassed multiple AWS services and outside protocols that I had little experience in as of then. Firstly, CORS has always been a headache for me, I still don’t fully understand it to this day. When deploying resources, the Console is a great tool for beginners. However, it includes too much hand-holding from AWS’ side. Enabling CORS for example on API Gateway is a single magical click, enabling it in a CloudFormation template is a whole another story. This was one of my biggest hurdles and sent me down deep rabbitholes and very old StackOverflow posts. Eventually an community member of the og-aws Slack channel helped my eyes to be guided into the right direction. I learned a lot about both API Gateway and it’s integration with Lambda in a REST API, as well as how HTTP headers work. Another issue I spent a whole afternoon troubleshooting was the IntegrationHttpMethod which turns out needs to be strictly POST — Lesson learned. **#7/9/10 Done!**

{{< amp-iframe
    sandbox="allow-scripts allow-same-origin"
    src="https://giphy.com/embed/3o7btPCcdNniyf0ArS"
    width="480"
    height="321"
    layout="responsive" >}}
    _If my Lambda function was a person. ⬆️_

DynamoDB is a great and seamless, fully managed NoSQL database. I have heard enough from genius AWS engineers and 3rd party companies praising the service that I fully trust it and turn to it whenever I can. There were no problems at this stage, just a small YAML snippet in the CloudFormation template and the database is deployed with a `PAY_PER_REQUEST` model due to the website’s nature. **#8 Done!**

When it came to Route53, things looked a bit more smooth. The only issue I’ve faced is with Cloudformation. When you want to create an alias entry into a hosted zone, you need to define the alias’ hosted zone as well. So when I wanted to create the alias for my main domain to be pointing to the CloudFront distribution, I needed to specify the distibution’s hosted zone. After some searching I realized that all CloudFront distirbutions have the same hosted zone ID. **#6 Done!**

## Stuff to still work on

_**UPDATE: See above**_

There are still some issues that need to be resolved. For example I would like to create a specific hosted zone record for the API Gateway so that the infrastructure becomes more decoupled (frontend-backend in this case). I would also like to add a Lambda function to put the visitorCount item into the DynamoDB database on initial deploy of the CloudFormation stack. One thing I left out (#13) was unit tests for the Lambda function. This is a matter of time and a bit of refactoring on the CI/CD pipeline, and the tests’ code is already finished actually.


{{< amp-iframe
    sandbox="allow-scripts allow-same-origin"
    src="https://giphy.com/embed/eFoKrBKieAEZ5fQIZY"
    width="480"
    height="321"
    layout="responsive" >}}


## Finishing words

With this Blog Post, I have completed #16. I have both enjoyed and learned a lot from this challenge. I would like to thank Forrest for creating it as I believe the very best way to learn anything is through actual projects — nothing beats hands-on experience. Forrest has asked us not to share the code for our projects and as much as I’d love to do that, I will honour his request and keep my repos private for at least a while. Of course I could not mention every single tiny detail and nuance learned while working on this, but rest assured all of them are in my mind-data-lake.

* * * 

✅ Check out the finished website at [heyitschris.com](https://heyitschris.com). If you’d like to get in contact with me, shoot an email at chris@heyitschris.com 👨🏻‍💻

_This post was written on the 23rd of June 2020, Updated on the 6th of July 2020._