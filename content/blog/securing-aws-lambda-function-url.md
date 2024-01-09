---
title: "Securing an AWS Lambda function URL with Amazon CloudFront and Lambda@Edge"
date: 2024-01-03T18:59:54+05:00
# weight: 1
# aliases: ["/first"]
tags: ["AWS Lambda","Aws Cloudfront", "Lambda Edge", "Serverless"]
author: Mudassir Hafeez 
# author: ["Me", "You"] # multiple authors
showToc: true 
TocOpen: false
draft: false
hidemeta: false
comments: true
# summary: "AWS Lambda function URL is a convenient way to expose your Lambda function to the public. Here, it is important to protect your function from unauthorized access. ..."
# description: "AWS Lambda function URL is a convenient way to expose your Lambda function to the public. Here, it is important to protect your function from unauthorized access. ..."
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
# searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false 
ShowRssButtonInSectionTermList: true
UseHugoToc: true 
cover:
    image: images/securing_a_lambda_function_url.png # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page

# editPost:
#    URL: "https://github.com/<path_to_repo>/content"
#    Text: "Suggest Changes" # edit text
#    appendFilePath: true # to append file path to Edit link

---

AWS Lambda function URL is a convenient way to expose your Lambda function to the public. Here, it is important to protect your function from unauthorized access. One way to do this is to use Amazon CloudFront and Lambda@Edge.

<!--more-->

> CloudFront is a content delivery network (CDN) that can be used to distribute your Lambda function URL to users around the world. Lambda@Edge is a feature of CloudFront that allows you to run Lambda functions at the edge of the network, closer to your users.

---

This blog shows how to use CloudFront and Lambda@Edge to protect a Lambda function URL by setting the authentication type to AWS IAM.

To protect your Lambda function URL with CloudFront and Lambda@Edge, you will need to:

   1. Create a CloudFront distribution and configure it to point to your Lambda function URL.
   2. Create a Lambda function that signs each request made to the Lambda function URL and adds the appropriate headers to it so that URL invocation is authenticated with IAM.
   3. Associate your Lambda function with the CloudFront distribution using Lambda@Edge.

## Step-1: Create a CloudFront distribution

Before creating a CloudFront distribution, you may first need to create a Lambda function, if it was not before, containing business code exposed to the frontend and configure a function URL with Auth type **AWS_IAM**. Please follow the mentioned links on how you can do this:

- [Create a Lambda function with the console](https://docs.aws.amazon.com/lambda/latest/dg/getting-started.html#getting-started-create-function)
- [Creating a function URL (console)](https://docs.aws.amazon.com/lambda/latest/dg/urls-configuration.html#create-url-console)

To create a CloudFront distribution, follow the link https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-web-creating-console.html or this is how a few options in web distribution behavior will look like:

{{< figure src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*oLNxOxuoAEDTvaWQQvpgyQ.png" align=center >}}

## Step-2: Create a Lambda function to check the authorization of each request

Now, you have to create a Lambda function that will sign each request by adding appropriate headers(`Authorization`, `X-Amz-Security-Token`, and `X-Amz-Date` ) before the request reaches to origin.

You can get the complete code for the request to be signed, on [github](https://github.com/mudassir-hafeez/auth_function_at_edge).

However, the following role policy would require for the Lambda@Edge function to invoke the target Lambda function.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "SidAllowInvokeFunctionUrl",
            "Effect": "Allow",
            "Action": [
                "lambda:InvokeFunctionUrl"
            ],
            "Resource": "arn:aws:lambda:us-east-1:account_id:function:target_function_name"
        }
    ]
}
```

In this policy, the only action that is allowed is `lambda:InvokeFunctionUrl`. The resource that the action is allowed on is the ARN of the target Lambda function.

## Step-3: Associate Lambda function with the CloudFront distribution using Lambda@Edge

Once a Lambda function is created to sign the request, then you have to deploy it to Lambda@edge. In order to do this,

- Choose `Deploy to Lambda@edge`
- Select CloudFront ID under `Distribution` to associate with function
- Select the `Cache behaviour` to use with the trigger
- Choose CloudFront event to `Origin request`
- Check Confirm deploy to Lambda@Edge
- Then click on `Deploy` button

Wait for the function to replicate. This typically takes several minutes.

You can check to see if replication is finished by [going to the CloudFront console](https://console.aws.amazon.com/cloudfront/v4/home) and viewing your distribution. Wait for the distribution status to change from **In Progress** back to **Deployed**, which means that your function has been replicated.

## Verify the solution
  1. Run the Lambda Function URL to retrieve the behavior:
  ```
  curl -v https://dummy34646c4dummybw3qa0helliv.lambda-url.us-east-1.on.aws/servers
  ```
  You should receive the following output. It’s confirming that direct access to the Lambda function URL without proper IAM authentication is forbidden:

  {{< figure src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*yZeRyEledI6uRGN3B_SIVw.png" align=center >}}
  
  2. Now run the “secured” URL to retrieve the behavior:
  ```
  curl -v https://dummy11111abcdef8.cloudfront.net/servers
  ```

  This time you should get an HTTP 200 OK with a dictionary as the result.
  
  {{< figure src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Gffd1MoZ5J7xbuD5ctZN1w.png" align=center >}}
  
Now that you have a basic idea of how to secure an AWS Lambda function URL with Amazon CloudFront and Lambda@Edge. If you need more advanced features like user authentication with [Amazon Cognito](https://aws.amazon.com/cognito/), request validation or rate throttling, consider using [Amazon API Gateway](https://aws.amazon.com/api-gateway/).

  
