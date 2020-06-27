---
title: "How it's made: This Blog"
date: 2020-06-27T16:12:39+02:00
draft: false
---

## How it's made: This Blog.

I've always wanted to have a blog. Putting my thoughts into writing has always benefitted me and I've definitely started paying more attention to do it as often as I can. After I completed [Forrest Brazeal's](https://forrestbrazeal.com/) [Cloud Resume Challange](https://cloudresumechallenge.dev/) and had my own website running on Serverless AWS, I thought why not extend it with a blog. I wrote some articles on my Medium page as well, but seeing #100DaysOfCode & #100DaysOfCloud take off, I decided it's time to make my own. This is the intorductory story, now let's get down to the *how*.

![Hugo Screenshot](/images/hugo-screenshot.png)

I knew I wanted to host everything as a static website on AWS. This ruled out Wordpress, which I really didn't want to go with because despite it being used for ~30% of the whole internet, is a ancient framework and frankly just too much for this project. The modern way is JAMStack, and the perfect candidate was [Hugo](https://gohugo.io/). Hugo is a static site generator with amazing features and a broad theme selection. It also allows you to write in `Markdown` which I so desperately love. The local setup is **super** easy and straighforward with their [Quickstart](https://gohugo.io/getting-started/quick-start/) guide. I won't repeat what they have on their website, rather I'll just give my 2 cents on the experience and a kind-of guide.

#### Let's get into it!
After installing Hugo with `brew` and creating the quickstart template, I looked around for a minimalist theme and found [Amperage](https://themes.gohugo.io/amperage/). I modified the `config.toml` a bit until I ended up with something I like. I don't have much experience in HTML&CSS and since Hugo was a brand new framework for me, I turned to YouTube for a ~20min tutorial which gave me everything I needed. In the process, I realized how **great** the JAMStack architecture is and how easy it is to deploy and create static websites.

Since I wanted this blog to be hosted straight in S3, I added the extra resources in my current CloudFormation template for my resume project, did a `git push` and my existing GitHub Actions CI/CD pipeline for the backend handled the rest. There is a seperate repository for this blog on my GitHub as well and it has it's own CI/CD workflow, but I wanted to deploy it from my machine first. This is where I scratched my head a bit, because the `hugo deploy` command doesn't currently support using the `--profile` flag like the AWS CLI, and my website lives in an Organizations sub-account. Fear none, `nano ~/.aws/credentials` and set the default access keys to the correct ones. I did a `hugo deploy --dryRun` at first as any cautious techie would do (*right?*) and it grabbed the instructions from my previously configured `config.toml` file. All in all, it was wonderfully easy to get everything up & running in less than an hour including the theme, the basic- and the deployment configs.

One gotcha I realized is that you _have to build the static site_ before you want to deploy. This is done by simply executing the `hugo` command in the project's root directory. This builds the static files, "compiling" everything into a finished product. The error you get if you don't do this says `the folder /public does not exist` - this is the reason why.

![hacker](https://media.giphy.com/media/YQitE4YNQNahy/giphy.gif)

#### GitHub Actions CI/CD for Hugo
Humans make mistakes all the time and we're pretty unreliable at doing tasks in a precise and repeatable manner. This is why I opted to use a GitHub Actions workflow to deploy this site from its repository to the S3 bucket as well. Hugo is very developer-friendly so there are clear instructions and a pre-made GitHub action by a community member. I grabbed the template, added my _deploy-to-s3_ job and `git push` that baby. Here's the `main.yml` file for the workflow:

{{< highlight "linenos=table" >}}

name: Deploy to S3 bucket

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          # extended: true

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
          
      - name: Hugo build
        run: hugo --minify

      - name: Hugo deploy static site to S3 bucket
        run: hugo deploy

{{< /highlight >}}

As you see, you should **never** commit access and secret keys to your repos. This is why you see `AWS_ACCESS_KEY_ID` in the *Configure AWS Credentials* part. GitHub offers `Secrets` which are encrypted environment variables that you can reference in your workflows and they are safe to use (even in public repos).

![itshappening](https://media.giphy.com/media/lkK7hFTOp1s4g/giphy.gif)

#### The AWS side
After all the backend was established - the CloudFront distribution, the CI/CD pipeline, the CloudFormation template - the only thing left was to add the `blog.heyitschris.com` entry into Route53. There is a deployment pipeline for every piece of the whole website at this point, so this step was nothing more than an extra resource in the CloudFormation template and `git push`. The resource deployed, this sentence finished and this blog's first iteration up & running publicly.

#### Problems after the deployment
Here is the part that makes this setup quite complicated on the backend to work. Since I am using CloudFront as a CDN in front of the bucket, using an `S3Origin` resource gives an **Access Denied** error on any of the `/posts/this-post` URLs. The reason I still haven't fully understood but the main point is that with Hugo, the distribution has to have the S3 Website **Endpoint URL** as an origin, meaning this: `${BlogWebsiteBucket}.s3-website-${AWS::Region}.amazonaws.com`. In order to do this, the bucket has to have all public access, since it's impossible to use an OAI with an endpoint URL. [This blog post helped me](https://lustforge.com/2016/02/27/hosting-hugo-on-aws/) figure it out, but I'll be writing one on here as well detailing the CloudFormation template. 

#### Closing thoughts
If you'd like to see the repo, you can [reach it here](https://github.com/what-name/heyitschris.com-blog)!