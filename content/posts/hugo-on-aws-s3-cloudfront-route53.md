---
title: "Hugo on AWS - S3/Cloudfront/Route53"
date: 2020-06-28T13:45:24+02:00
draft: false
---

## How to host a Hugo static site on AWS Serverless with CloudFormation.

I built this blog with Hugo and wanted to host it on my already existing AWS S3 static site infrastructure. The problem was however that the main page loaded no problem, but anything in `/posts/this-post` would get an `AccessDenied` error. While I haven't figured out _why_ this problem exists, I did make it work and now I'm sharing the setup here since there is no clear-cut guide on Hugo for this setup.

#### Notes
You can not use an `OAI` with Hugo. You need to use the **Endpoint URL** of the S3 bucket as the origin for your CloudFront distribution. An endpoint URL looks like this: `myblogbucket.s3-website-us-east-1.amazonaws.com`. Notice the region, this is the endpoint URL of your bucket, and you can get this from the `Properties->Static Website Hosting` submenu in the console inside the origin bucket.

### The resources
Here are the most important resources and their properties with a YAML CloudFormation template.

#### The blog bucket itself
I added the CORS configuration, you might not need it. [See AWS CloudFormation S3 reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html)
{{< highlight "linenos=table" >}}
BlogWebsiteBucket:
  Type: AWS::S3::Bucket
  Properties:
    BucketName: "mywebsite-blog"
    WebsiteConfiguration:
      IndexDocument: index.html
    CorsConfiguration:
      CorsRules:
        - 
          AllowedMethods: 
            - GET
            - HEAD
          AllowedOrigins: 
            - "*"
          AllowedHeaders: 
            - "*"
{{< /highlight >}}

#### The blog bucket policy
As I said you **can not use an OAI** with your CloudFront distribution here because you need the endpoint URL of the bucket for Hugo to be able to do its job. For this reason, you need to give public read access to the bucket. While this is not a perfect solution, it's the one that worked for me and I'm willing to sacrifice this much security. (if you know a better way of doing this, please hit me up on [Twitter](https://twitter.com/chris_the_nagy)) [See AWS CloudFormation S3 bucket policy reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-policy.html)
{{< highlight "linenos=table" >}}
BlogWebsiteBucketBucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      PolicyDocument:
        Id: PublicAccessPolicy
        Version: 2012-10-17
        Statement:
          - Sid: PublicReadForGetBucketObjects
            Effect: Allow
            Principal: "*"
            Action: s3:GetObject
            Resource: !Sub "arn:aws:s3:::${BlogWebsiteBucket}/*"
      Bucket: !Ref BlogWebsiteBucket
{{< /highlight >}}

#### The CloudFront distribution
This is the most important part and I spent most of the troubleshooting time on this. The important part here is that we're not using an `S3Origin` but a `CustomOriginConfig` pointing to the endpoint URL of the S3 bucket. It's important to set the `DefaultRootObject: index.html` on both the distribution and the origin S3 bucket, otherwise your browser will try to _list_ the contents of your bucket (and prefix) instead of serving you the page. [See AWS CloudFormation CloudFront reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-cloudfront-distribution.html)
{{< highlight "linenos=table" >}}
BlogCloudFrontDistribution:
  Type: AWS::CloudFront::Distribution
  Properties:
    DistributionConfig:
      Origins:
      - Id: !Sub "${BlogWebsiteBucket}-origin"
        CustomOriginConfig: 
          OriginProtocolPolicy: http-only
          HTTPPort: 80
          OriginSSLProtocols: 
            - TLSv1
            - TLSv1.1
            - TLSv1.2
          HTTPSPort: 443
        DomainName: !Sub "${BlogWebsiteBucket}.s3-website-${AWS::Region}.amazonaws.com"
      Enabled: true
      Comment: Static blog website distribution
      DefaultRootObject: index.html
      Aliases:
        - blog.yourwebsite.com # Change this to your domain!
      DefaultCacheBehavior:
        DefaultTTL: 1800
        MaxTTL: 14400
        Compress: true
        AllowedMethods:
          - HEAD
          - GET
        CachedMethods:
          - HEAD
          - GET
        TargetOriginId: !Sub "${BlogWebsiteBucket}-origin"
        ForwardedValues:
          QueryString: 'false'
        ViewerProtocolPolicy: redirect-to-https
      PriceClass: PriceClass_100
      ViewerCertificate:
        AcmCertificateArn: !Ref DomainCertificate
        SslSupportMethod: sni-only
{{< /highlight >}}

#### The Route53 Entry.
Pay attention to the `HostedZoneId` of the `AliasTarget`! Took me a while to figure out that **all** CloudFront distributions live in the same hosted zone `Z2FDTNDATAQYW2`. [See AWS CloudFormation Route53 RecordSet reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-route53-recordset.html)
{{< highlight "linenos=table" >}}
 DNSRecordBlog:
    Type: AWS::Route53::RecordSet
    Properties:
      Name: !Sub "blog.${WebsiteDomainName}"
      Type: A
      # Refers to the HostedZone resource that I didn't include here
      HostedZoneId: !Ref HostedZone
      AliasTarget:
        DNSName: !GetAtt BlogCloudFrontDistribution.DomainName
        HostedZoneId: Z2FDTNDATAQYW2
{{< /highlight >}}

#### ACM Cert
And finally a little extra, the ACM certificate for both the main `mydomain.com` and the wildcard `*.mydomain.com`. The little _gotcha_ here is that the main cert is the wildcard, and the main domain is only an alternative name. [See AWS CloudFormation Certificate Manager reference.](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-certificatemanager-certificate.html) 

🚨Important to remember here that your CloudFormation stack will not proceed until you verify your domain via the email ACM sends to the adress that's registered with the domain!

{{< highlight "linenos=table" >}}
DomainCertificate:
  Type: AWS::CertificateManager::Certificate
  Properties:
    DomainName: !Sub "*.${WebsiteDomainName}"
    SubjectAlternativeNames:
      - !Ref WebsiteDomainName
      - !Sub "*.${WebsiteDomainName}"
    DomainValidationOptions:
      - DomainName: !Ref WebsiteDomainName
        ValidationDomain: !Ref WebsiteDomainName
    ValidationMethod: EMAIL
{{< /highlight >}}

### Closing Thoughts
I like [Hugo](https://gohugo.io/) a lot. It's a very easy and developer friendly way of creating and updating static sites and blogs. This setup is not the "usual" way of hosting a Hugo site but it is an option and works well in my opinion while maintaining ultra low cost (my whole website costs me less than 5 cents a month).

If you have any thoughts, questions or recommendations, hit me up on [Twitter](https://twitter.com/chris_the_nagy) or at contact@heyitschris.com.

_This post was written on 28th of June 2020._