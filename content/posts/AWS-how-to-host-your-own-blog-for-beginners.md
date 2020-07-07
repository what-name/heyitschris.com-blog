---
title: "AWS - How to Host Your Own Blog (for beginners)"
date: 2020-07-07T15:05:46+02:00
draft: false
---

## AWS - How to Host Your Own Blog (for beginners)

Having a personal blog is great for a multitude of reasons. With the [#100DaysOfCloud](https://100daysofcloud.com) challenge, it is especially useful for both detailing what projects you've done and how, as well as sharing your knowledge with other learners or with potential employers. In this post I'll walk you through all of the steps on how to create one and host it on AWS. You will of course need an AWS account for this which if you don't have already, you can enroll in the [free tier](https://aws.amazon.com/free/). While the free tier offers a lot of wiggle room, hosting your blog will cost just about $1 per month, or completely free if you don't want to have a custom domain name. I will explain all the services and their prices in detail as we go further into the setup. If you have difficulties completing the sections, always try to Google the error first and try to solve it on your own for 30 minutes. If you are still having trouble, you can hit me up on [Twitter](https://twitter.com/chris_the_nagy) or post your question with the _#100DaysOfCloud_ hashtag. I will also assume that you have at least some working knowledge of using a terminal, if not, [here's a great tutorial from FreeCodeCamp.org](https://www.freecodecamp.org/news/how-you-can-be-more-productive-right-now-using-bash-29a976fb1ab4/). Let's roll up those sleeves and get started!

{{< amp-iframe
    sandbox="allow-scripts allow-same-origin"
    src="https://giphy.com/embed/xT5LMJPvVaukDXS4bC"
    width="480"
    height="321"
    layout="responsive" >}}

## What we'll use
These are the things or services that you'll learn about with this project:
- AWS CloudFront, which is a content distribution network. It will ensure fast load times for people visiting your blog, no matter where they are physically located.
- AWS S3, which is object storage. This is where you'll store your website's files.
- AWS CLI, which is the command line interface for AWS resources. You'll use this to upload your blog's files to S3.
- (Optional) AWS Route53, which will be handling the custom domain. Without a custom domain, your site will be still reachable at a URL such as this: `j39gh3f8.cloudfront.net`. It's not required to have your own domain, but highly encouraged. You can use a domain you already have or you can buy one from AWS itself as well.

### ⚠️ Quick Note
If you want to create your custom-made website instead of a blog, ignore all the parts where I talk about Hugo or static site generators, and simply upload your website's static files into your bucket instead.

### Creating your blog's frontend
Since the site will be a static one, I will explain how to use the [Hugo](https://gohugo.io) framework, but if you are more comfortable with Jekyll, Gatsby or VuePress - you are welcome to use those as well. What's important at the end of the day is to have a simple `index.html` that routes to your posts. I like Hugo, it is reliable, easy to use and there are a ton of pre-built themes for it - as well as some _gotchas_ when using it with CloudFront. You are also welcome to create your own blog solution, as long as it's purely static files.

#### Hugo
To create a Hugo static blog, head over to [GoHugo.io' QuickStart guide](https://gohugo.io/getting-started/quick-start/). For MacOS and Linux users, installing Hugo is a single command with [Brew](https://brew.sh/); `brew install hugo`. For Windows users, see the [Install on Windows](https://gohugo.io/getting-started/installing/#less-technical-users) section. After the installation was successful, execute the `hugo version` command in the terminal to see if it's working. If it spits out the version, you're good to go! If you get an error, refer to the [install help](https://gohugo.io/getting-started/installing/).

![1_S3_0.png](/images/aws-host-your-blog/1_S3_0.png)

Lets create our site!
- Open up your terminal where you want to create your site's folder, and execute the `hugo new site my-blog` command. This will create a folder called `my-blog` which will host all of your blog's files.
- Execute `cd my-blog` to change to that directory.
- Initialize a local git repository with `git init`. If you don't have git, you can install it from [here](https://git-scm.com/downloads).
- You need a theme for your blog! Let's add a basic one with `git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke`. This will add the `ananke` theme to your project in the `themes` folder. You can browse for different themes on [Hugo's website](https://themes.gohugo.io/).
- The theme is now there, but Hugo doesn't know about it yet! Let's execute the `echo 'theme = "ananke"' >> config.toml` command to add our new theme to the project's config file. If you want to change your theme later, you can simply edit the `config.toml` file and change it to your new theme's name.
- To create your first blog post, execute the `hugo new posts/my-first-post.md` command. This will create a file called `my-first-post.md` (written in [Markdown](https://www.markdownguide.org/getting-started)) in the `content/posts` folder. You can open this file and paste some _lorem ipsum_ into it or simply write anything you want it there (for example "hello there, general kenobi"😉). In the header section of the post's file, you see multiple fields such as name, date and so on. The important one I need to mention is `draft`. If draft is set to `true`, your post will not be visible when we later generate the site itself. It's important to set this to `true` for now in order to save yourself from some headaches.
- Are you ready to see your new blog? While still in your project's directory, execute the `hugo server -D` command. This will spin up a tiny webserver in the background and make your new website reachable locally. Navigate to the link it gives you - which is [http://localhost:1313](http://localhost:1313) - and ponder upon your creation! If you make changes to your site locally, the tiny-webserver will automatically refresh itself, no need for a restart. You can stop the hugo server with `ctrl+c`.
- To create the actual blog's static sites (which we'll upload to S3 later), execute `hugo -D`. This will create a folder called `public` in your project's directory and put the required files there. Note that every time you make changes to your blog such as creating a new post or changing your theme, you will need to run the above command to update the static files in the `public` folder. _Posts with "draft: true" do not get put into that folder!_

## AWS
Log into your AWS Console with an IAM user that has `AdministratorAccess` privileges. If you are currently logged in with your root user (you log in with your email + password), I would urge you to create a seperate IAM user for everyday use. It is **very important** not to use your root user for anything, not this project and especially not in a work environment. [Here's an amazing guide from AWS](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html) on how to do that.

![1_S3_0-login.png](/images/aws-host-your-blog/1_S3_0-login.png)

### S3
Firstly you need to create an S3 bucket. Go to the S3 console and click on the button "Create Bucket".
![1_S3_1.png](/images/aws-host-your-blog/1_S3_1.png)
In the following screen you need to fill out your bucket's name. The name of your bucket has to be **globally unique** meaning that no two buckets in the whole of AWS can have the same name. It doesn't actually matter what the name of your bucket is, go with something that you'll know, for example *"your-name-blog"*. The region doesn't really matter in our case either (because we will interact with CloudFront only), so choose one that you're familiar with, have other resources in or the one that's the closest to you. After you're done, click Next.
![1_S3_2.png](/images/aws-host-your-blog/1_S3_2.png)
Since this tutorial is aimed towards beginners, we will not use any of the encryption, versioning or logging functions. Click Next to go to "Set Permissions". On this page, it is important [for Hugo] that you **do not** block all public access, I'll explain later. Next - Review if everything looks good and hit "Create Bucket".
![1_S3_3.png](/images/aws-host-your-blog/1_S3_3.png)
Click on your bucket's name in the console (clicking on its surrounding field only brings up a windows with information) to get into it. Go to Properties -> Static Website hosting.
![1_S3_4.png](/images/aws-host-your-blog/1_S3_4.png)
Click on "Use this bucket to host a website", type in `index.html` as the root object and hit save. The link on the top of that window will be the access URL of this bucket's website. Every static website hosting enabled bucket has the same URL structure; `<bucket-name>.s3.amazonaws.com` but they are also accessible with their region `<bucket-name>.s3-us-east-1.amazonaws.com`. If you ever forget your bucket's website URL, just remember this simple rule.
If you go to your bucket's URL right now however, you will get a *giant* 403 Error. For this, you'll need to add a **bucket policy**. A bucket policy looks the exact same as an IAM policy but is attached to and influences a bucket's behaviour instead of a user or role. Because of Hugo and CloudFront, we will need use a *public* bucket policy for this project. Usually allowing complete public [read] access to a bucket is a [very bad idea](https://www.hackread.com/unprotected-aws-bucket-exposes-50-gb-of-financial-giants-data/), but for our case it's required and not a security threat. You can learn more about bucket policies [here](https://cloudacademy.com/blog/amazon-s3-security-master-bucket-polices-acls/). Below is the bucket policy that we will use. Replace the `<your-bucket-name-here>` with your bucket's name, and don't forget to click "save"!

![1_S3_6.png](/images/aws-host-your-blog/1_S3_6.png)

- Action defines what actions are allowed/denied, in our case `GetObject` which means reading the objects in the bucket.
- Effect can be either allow or deny. You can explicitly deny for example specific actions while allowing others.
- Resource is what resource this policy applies to. In our case it is this bucket, followed by a `/*` meaning all the objects in the bucket. Without the `/*`, the policy applies to the bucket itself. (This small info saves you hours in troubleshooting, trust me)
- Principal means _who_ the policy applies to. It can be an IAM user, a service or in our case `*` means everybody on the public internet.
- ID and Sid can be anything you want, it is for you to identify or describe the policy.
- Version is the constant value `2012-10-17`, don't change that.
{{< highlight "linenos=table" >}}
{
  "Id": "MyBlogBucketPolicy",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPublicRead",
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::<your-bucket-name-here>/*",
      "Principal": "*"
    }
  ]
}
{{< /highlight >}}

## Upload your blog to S3
This is the time where I like to check if components that I'm building work properly, so that when a problem comes up later I know that it worked in an isolated environment. For this reason, we're going to upload the blog's static files to S3 now to see if it works (it will😉). Unfortunately it is not allowed to upload folders through the S3 console, so as a learning opportunity we will use the CLI [Version 2]. If you don't have the AWS CLI installed yet, [here is the guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html). Come back after having it installed and working, I'm not going anywhere. The reason why I'm not detailing how to install it now is that you will most likely need to reference and search for a million things when building a project. Don't be afraid to Google (or rather [Duck.com](https://duck.com)) things you don't know or errors you've never seen. **Googling is _very much_ encouraged**.

### Configure the CLI
*If you already have your AWS CLI configured, you can skip this part.*
After you've installed the CLI, you need to configure it. Firstly, go to "IAM" on the AWS console. In the left sidebar click on "Users", then click on the name of your current user and your screen will look like the one below.

![1_S3_7.png](/images/aws-host-your-blog/1_S3_7.png)

Now "Security Credentials" and then "Create Access key" which will give you an Access Key and a Secret Access Key. **It is very important never to share, commit or otherwise let these out of your hand!** If somebody has your access keys, they have access to all the resources that user would have access to, including the ability to create new ones or destroy anything in your account. Note these down for now in a **secure place**.

![1_S3_8.png](/images/aws-host-your-blog/1_S3_8.png)

The next step is to configure the CLI itself. For this, open a terminal and type in `aws configure`. It will ask for your access key, your secret key, a default region and a default output format. Paste in your access keys, choose the region where your S3 bucket is located and for the default output format type in "json" or "table". After this is done, the magic "who am i" command for AWS is `aws sts get-caller-identity`, which will return your username if all went well.

![1_S3_9.png](/images/aws-host-your-blog/1_S3_9.png)

### Upload your blog to the S3 bucket
Here comes the fun part where you'll bridge the gap between the local development of your blog, and actually putting it on AWS😎

{{< highlight "linenos=table" >}}
# Navigate to your Hugo blog project's directory in your terminal
cd <your-path>/my-blog
# (Just for safe measure again) Build your site's static assets
hugo
# Sync all the files from the public directory to your S3 bucket
aws s3 sync public s3://not-john-cena-blog
{{< /highlight >}}

If any of the commands above would give an error, don't panic - Errors are great! They (usually) give you a very precise reason on why they failed. Read the error message carefully and think where it could have gone wrong. In case you did _not_ get any error messages, that's also great! That means your whole website was successfully uploaded to your S3 bucket. Go back to your bucket in the AWS console and see if your files are indeed in there. If they are, go to your bucket's website URL (`<your-bucket-name>.s3.amazonaws.com`) and voilà - your blog is up and running! (Although quite empty right now, but that's for later)

## Quick Note
1. Every time you make changes to your blog locally, you will have to build it and upload it to your bucket manually. Changes you make locally will not be reflected on your website unless you update your bucket's contents as well.
2. You can very well stop here, take a break and continue later. Depending on your current skillset, having come this far is already an achievement, go drink some water as a prize haha!
3. Your main page will work well, but your posts will not be available right now, their link will redirect to "example.com". You need to manually edit your site's `config.toml` file under `baseURL` to reflect your site's URL as Hugo is a light _static site generator_, it does not know which URL it is being loaded from.


### Where you're at right now
At this point, you have an S3 bucket with a public read policy. You have a working AWS CLI that allows you to interact with your AWS resources from the command line. You also have your blog's Hugo project ready to be filled with blog posts. You know how to:
- Create a Hugo blog
- Create an IAM user on AWS and it's access keys
- Create a an S3 bucket, add a policy to it and enable static website hosting

## CloudFront
If you were to leave your website running purely on S3, it would cause some significant loading time delays for people visiting it from the other side of the world. Every single packet has to travel thousands of miles (kilometers) to get to its destination and that increases load times significantly. This is why we'll utilize the power of CloudFront to serve your website from AWS' ~100 edge locations. Edge locations are small data centers where Amazon has servers of their own that serve up your static files. When a user visits your site, they will automatically be routed to the server that has the fastest response to reduce load time for your users.

### Create a CloudFront distribution
In the AWS Console visit CloudFront and click on "Create Distribution". You will be offered to chose either "Web" or "RTMP", choose Web.

![1_S3_10.png](/images/aws-host-your-blog/1_S3_10.png)

On the next screen, you will be confronted with a ton of settings to choose from. We don't need most of them but there are some important ones you need to pay attention to. Firstly, the _Origin Domain Name_ is where your website is currently hosted, the S3 URL. This is the big _gotcha_ here with Hugo because Hugo does not work with a `<your-bucket>.s3.amazonaws.com` origin. It **has to include the region as well**. So for this, copy your current S3 website's link in the form of `<your-bucket>.s3-website-us-east-1.amazonaws.com`, that `us-east-1` (region) is the important part here. Next up, choose "Redirect HTTP to HTTPS" in the Viewer Protocol Policy, this will ensure that all the traffic will be utilizing TLS encryption, which is a must-have in 2020. There are some changes you will need to make to this if you want to use your custom domain, but we'll come back to that later in the Custom Domain section. Notice that you have a little "i" icon next to every setting, you can click those and get a brief overview of what those settings mean.

![1_S3_11.png](/images/aws-host-your-blog/1_S3_11.png)

Next up under "Distribution Settings" there is the "Price Class". This influences how many edge locations your site will be stored on. There more edge locations, the pricier it will be of course and for this project it's perfectly fine to go with just the bare minimum - US, Canada and Europe. If you are from Africa or Asia, you can choose the middle option as well, it won't break the bank at this scale. The next very important detail is "Default root object", which just like in S3 has to be `index.html`. The default root object is the file that will be served up by default upon visiting your site.

![1_S3_12.png](/images/aws-host-your-blog/1_S3_12.png)

We're almost finished! At the very bottom there is a "Comment" field which you can type into "My new blog!", but don't have to. It helps a lot with identifying which distribution does what and is generally a great idea to give names and descriptions to your resources. At the very bottom under "Distribution state", make sure it is set to _"enabled"_ and hit _Create Distribution_ in the bottom left corner. Now you'll have to wait around ~5 minutes for CloudFront to take all your assets from your S3 bucket and distribute it to a dozen servers all around the world. How amazing is that?! You can see if it finished yet in the "Status" field as seen below. You can later come back and modify the settings of your distribution by clicking on the distribution's ID (colored in blue). You can also refresh the page with the refresh button to see _"is it up yet? is it up yet?"_

![1_S3_13.png](/images/aws-host-your-blog/1_S3_13.png)

Now all you have to do is go to the URL that is shown under "Domain name" on the CloudFront console next to your distribution's ID as seen in the screenshot above. If all went well, you're shown your flashy new blog in all its glory ith blazing fast load times! 

You'll notice however that the link to your blog post does not redirect to the post itself, instead it redirect to `example.com`. This is Hugo's quirk and it's defined in your Hugo project's directory in the `config.toml` file under `baseURL`. This is the time where you need to decide if you want to have a custom domain or not. If you're fine with the CloudFront domain, you have to modify this value to your distribution's domain name (the one you just visited). If you want to have your own custom domain name, you can modify if to the domain you just bought (or already have), build the project with the command `hugo` and sync the contents of your public folder to S3 again. An important note here is that since CloudFront is a _cache_, changes you make in your S3 bucket will not be reflected right away on your `yourid.cloudfront.net` website as explained below.

![1_S3_14.png](/images/aws-host-your-blog/1_S3_14.png)

### TTL and cache invalidation
TTL means time-to-live, as in how long a file will be cached before it is sourced from the origin (S3) again. It is a very delacate negotiation between browsers and servers and must always be respected. On the other hand, if you want to see the changes you make on your blog instantly (as I do since a new blog post has to be available right away), you can create a so called "invalidation". This is an instruction essentially to CloudFront that it should go and retrieve the new files from your origin and update its cache forcefully. In general, you should very rarely choose to opt for an invalidation in a production environment and always rely upon / edit the TTL. In our case, we will use invalidations but note that they do cost a couple cents per million objects, so don't go crazy with them, and you only ever need it if you changed your bucket's contents.

To create an invalidation, click on the distribution ID of your CloudFront distribution (colored in blue), then go to "invalidations" -> "create invalidation" and put in the objects you want to invalidate. At this scale, I usually go with a `"*"`just for safe measure (which invalidates everything), but again, this is very much not best practice in production scale setups. After you create an invalidation, it will show up right there in the console and that it's "in progress". After it is done, visit your CloudFront link again to see the changes. This is why it's important to do make sure that your local development of your Hugo project is working, so that you don't have to invalidate the cache too often but only when you create a new post. 

![1_S3_15.png](/images/aws-host-your-blog/1_S3_15.png)

## Custom Domain
Do you already have a domain want to buy one now? If you already have one with a 3rd party provider such as [NameCheap](https://namecheap.com), [GoDaddy](https://godaddy.com) or [NameSilo](https://namesilo.com), you are welcome to use that. If you don't currently have one, you can buy it with any of those providers or with AWS directly. I bought mine with AWS but note that it's usually cheaper to buy with a 3rd party provider, especially because they offer $1 domains such as `.xyz`. If you buy one with AWS, a "hosted zone" in Route53 will be created for you automatically. If not then we will create a custom hosted zone right now and point the domain's DNS settings to the hosted zone. Note that a hosted zone will cost you ~$0.40 per month (not included in the free tier).

To do this, go to Route53 (Services drop-down menu) and click on "Create Hosted zone". You will be prompted to type in your hosted zone's domain name, which needs to be the same as the domain you just bought (or already have).

![1_S3_16.png](/images/aws-host-your-blog/1_S3_16.png)

This will give you NS and SOA records. The NS record stands for "name servers" and these are the servers that will translate `myblogdomain.com` to an IP adress inside AWS. You will need to copy _all four of these records_ and paste it into your 3rd party provider's DNS nameserver settings for your domain name. Since every provider is different, [here's an example guide from GoDaddy](https://au.godaddy.com/help/change-nameservers-for-my-domains-664). The main point here is to use your _custom name servers_ which are in fact AWS'. After you've changed these successfully, it will take about 5-48 hours for the changes to be active. This is beacuse [DNS](https://www.cloudflare.com/learning/dns/what-is-dns/) is a global and distributed service and it takes some time for all the servers involved to propogate the change. Note the dots at the end of the name servers' URL, they are very important and *usually* have to be included on your provider's end as well.

![1_S3_17.png](/images/aws-host-your-blog/1_S3_17.png)

After you've done this, you will need an SSL certificate. Not to worry, AWS offers them at no charge whatsoever! To do this, go to your CloudFront distribution and under "General" click "Edit". It will bring you back to all those settings you saw before when creating the distribution. The important part here is the "Alternate domain names" and "SSL certificate". Into the Alternate domain names field, type in your `mydomain.com`, as well as `www.mydomain.com` in two seperate lines underneath each other. To create an SSL certificate, choose the "Custom SSL certificate" option and click on "Request or Import a certificate with ACM". The process is self-explanatory, create your certificate then choose it back on the CloudFront settings page. It's important here that when you create a certificate, you will need to confirm ownership of your domain. This will be usually in the form of an email to the email adress that you provided in the domain's records when you bought it. Simply click the link you get in the email and confirm your certificate.

![1_S3_18.png](/images/aws-host-your-blog/1_S3_18.png)

After this is done (and it's very importanht that you fill out both the alternative domain names and create a certificate), go back to the Route53 and click on your `myblog.com` hosted zone. Here you will create a new record with the "Create record" button. See the screenshot below for how your fields should look like. Choose "yes" to the Alias as this will allow you to choose the CloudFront distribution directly (as seen on the screenshot). If you do not provide the alternative domain name and certificate to your distribution, it will **not** show up in this list. Choose your CloudFront distribution as your Alias Target, keep the top text field empty, no health checks required here and hit "Create". As a small challenge, try to create another record that redirects `www.myblog.com` to `myblog.com` on your own😉 (tip: use an alias here as well, pointing to your main domain).

![1_S3_19.png](/images/aws-host-your-blog/1_S3_19.png)

If all went well, (after a couple hours because of DNS), you will be able to hit [myblog.com](myblog.com) and be presented with your beautiful new blog! Now all it needs is more posts! I would encourage you to write one about how you created your own blog in order for your knowledge to really solidify, and explain how you overcame all the challenges you were faced with.

## Whoop Whoop! You have a blog now!
When you make changes locally to your blog, don't forget to push them to your S3 bucket, then create a CloudFront invalidation in order for it to become visible right away. You don't have to invalidate the cache, but it will take a while for your blog to display your new post if you don't do it.

### The Hugo "gotcha"
The reason why you can't use an origin URL for CloudFront like `myblog.s3.amazonaws.com` is a problem with how Hugo hides its `index.html` in the URL in contrast to S3 and CloudFront. I have not found the exact underlying reason yet, but the working workaround is to use the S3 endpoint in CloudFront that includes the region as well.

## Finishing words
After you created your website, feel free to hit me up on [Twitter](https://twitter.com/chris_the_nagy) to show me what you made! I'm always happy to check out others' creation and work. Similarly, if you have any trouble at any of these steps, don't hesitate to reach out to me or tweet your question with the #100DaysOfCloud hashtag. The community is great and me or somebody will help you out!

Happy blogging!😎

_This post was published on the 7th of July, 2020._