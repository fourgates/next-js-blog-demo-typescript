---
title: "A No-Nonsense Guide To AWS Cloud Development Kit (CDK)"
date: "2020-01-02"
---

In my [previous post](https://blog.phillipninan.com/5-reasons-to-use-aws-cdk), I gave you **five** reasons why you should use the AWS Cloud Development Kit (CDK)! Today, I will give you a high-level overview of what the CDK is and how to use it. This guide assumes you already have an AWS account, have a basic understanding of some AWS resources, and have `npm` installed.

## Table of Contents
- [Key CDK Terms](#key-cdk-terms)
- [AWS Construct Library](#aws-construct-library)
   - [L1](#l1)
   - [L2](#l2)
   - [L3 / Patterns](#l3)
- [Stacks vs Constructs](#stacks-vs-constructs)
- [How Many Stacks Should My Application Have?](#how-many-stacks-should-my-application-have)
- [Getting Started - CDK Example](#getting-started-cdk-example)
   - [Install AWS CLI](#install-aws-cli)
   - [Install AWS CDK](#install-aws-cdk)
   - [Create the app](#create-the-app)
   - [Build the app](#build-the-app)
   - [List stacks in your app](#list-stacks-in-your-app)
   - [Add an Amazon S3 bucket](#add-an-amazon-s3-bucket)
   - [Synthesize App](#synthesize-app)
   - [Deploy App](#deploy-app)
   - [Modify App](#modify-app)
   - [Destroy App Resources](#destroy-app-resources)
- [Next Steps](#next-steps)
- [Writing Your Own Constructs](#writing-your-own-constructs)
- [Conclusion](#conclusion)

🐦 Follow me on [Twitter](https://twitter.com/ninan_phillip) if you would like to see more content like this! 🐦

## Key CDK Terms
- **App** - You can think of a CDK app as the **composition of all the resources** needed for your application. This can deploy your entire AWS infrastructure.
- **Stacks** - A stack in AWS CDK is the same as in AWS CloudFormation. You can think of stacks as **groups of resources** needed for your application. This is defined as a single unit of deployment.  An app can be composed of one or more stacks. 
- **Constructs** - You can think of constructs as an **individual object** in a stack. These objects can be composed of one or many AWS resources. Constructs are the basic building blocks of an app or stack. Each stack is composed of constructs. 
- **Etc** - I have listed the most important concepts. You can read more on the [AWS documentation](https://docs.aws.amazon.com/cdk/latest/guide/core_concepts.html)!

>Here is a useful diagram explaining these relationships.
![AppStacks.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617136509362/C7AdUxn5w.png)

## AWS Construct Library
AWS provides an entire [library full of constructs](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html) that are ready to use. They are very well documented and provide no-nonsense example. Let's look at the different types of constructs. 

### L1
Some constructs provide a single resource such as an Amazon S3 bucket or a Lambda function. These are known as **level-one** or **L1** for short. **These constructs are very basic and must be manually configured. ** They will have a `Cfn` prefix and correspond directly to AWS CloudFormation specifications. New AWS services are supported in the AWS CDK as soon as AWS CloudFormation does. [`CfnBucket`](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.CfnBucket.html) is a good example. It represents an Amazon S3 bucket where you MUST **explicitly configure ALL the properties. **
```
const bucket = new s3.CfnBucket(this, "MyBucket", {
  bucketName: "MyBucket"
});
```

### L2
The next level, **curated** or **L2**, provides constructs with common boilerplates and glue logic. **These will come with convenient defaults and reduces the amount of knowledge you need to know about them.** They will typically encapsulate their corresponding L1 modules. A good example is [`s3.Bucket`](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html). This class will create an Amazon S3 bucket with default properties and methods such as [`bucket.addLifeCycleRule()`](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html#add-wbr-lifecycle-wbr-rulerule), which adds a lifecycle rule to the bucket.
```
new s3.Bucket(this, 'MyFirstBucket', {
  versioned: true
});
```

### L3
While others, **called high-level** or *patterns*, allow you to provision resources that are commonly used together. **Construct *patterns* are designed to help you provision multiple resources based on common patterns with a limited amount of knowledge in a concise manner. ** `s3deploy.BucketDeployment` is a great example of a high-level construct. The L3 construct takes a bucket as an argument and uploads your code.
```
const websiteBucket = new s3.Bucket(this, 'WebsiteBucket', {
  websiteIndexDocument: 'index.html',
  publicReadAccess: true
});

new s3deploy.BucketDeployment(this, 'DeployWebsite', {
  sources: [s3deploy.Source.asset('./website-dist')],
  destinationBucket: websiteBucket,
  destinationKeyPrefix: 'web/static' // optional prefix in destination bucket
});
```

## Stacks vs Constructs
Stacks and constructs can both be used to provision groups of AWS resources. Your cloud application can have many stacks leveraging different levels of constructs. The key difference is that stacks are deployed independently. **Therefore, you should decouple constructs into stacks if you would like to isolate different layers of your cloud infrastructure.** For example, you could compose multiple constructs into a`DevStack`. This stack can be configured for a development environment while having a different composition for production.

## How Many Stacks Should My Application Have?
There is no hard and fast rule to determine this. I asked an [AWS employee](https://twitter.com/nathankpeck) this question. He gave me a great response! 

A stack is just a group of resources. One giant group of resources will inhibit your ability to tear down resources independently of your other resources.
Dividing your resources into smaller groups can make it easier to manage and roll back/forward changes.

He recommended always putting your networking and database layer (stateful resources) into a base stack. Next, you should put your application (stateless) resources into one or more stacks. This way you can tear down the app and recreate it while leaving up your network and database layer intact.

A second recommendation came from another [AWS employee](https://twitter.com/realadamjkeller). He noted the parallels between [Conway's law](https://en.wikipedia.org/wiki/Conway%27s_law) and deploying stacks. 

>"Any organization that designs a system (defined broadly) will produce a design whose structure is a copy of the organization's communication structure."
— Melvin E. Conway

Essentially, you may want to break down your stacks to mirror how your teams operates. At the end of the day, build in a way that makes the most sense for you and your team.

##  Getting Started - CDK Example
> **TLDR** - Here is a link to my [Github Repo](https://github.com/fourgates/hello-cdk) with the fulling working example. 

### Install AWS CLI
> First, you need to have the [AWS CLI installed and configured](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html). The CDK uses the same credentials as AWS CLI.

### Install AWS CDK
> Next, install the AWS CDK Toolkit using `npm`
```
npm install -g aws-cdk
# verify installation
cdk --version
```

### Create The App
> Create a directory and initialize a project with your language of choice. I love TypeScript! 
```
mkdir hello-cdk
cd hello-cdk
# init project
cdk init app --language typescript
```

## App Structure
> This is what the app file structure should looked like. A few notes. `bin` folder is the entry point. This is where the app is initialized. `cdk.out` directory is where the CloudFormation template gets generated. Lastly, `lib` is where all of your stacks and constructs will live.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617596519813/CwvPI-ICs.png)

### Build The App
> Next, you will need to build your project whenever you make coding changes.
```
npm run build
# or you can watch
npm run watch
```

### List Stacks in Your App
> Verify everything is working and you see the stacks you are expecting. You should only have one stack at this point. 
```
cdk ls
```

### Add an Amazon S3 bucket
> Next, I will show you how to add a construct. We will add a simply S3 bucket. You need to add dependencies for each package you wish to use. Please make sure all of your cdk package versions match or you can run into trouble!
```
npm install @aws-cdk/aws-s3
```

Update the stack inside the `lib` folder (~`hello-cdk-stack.ts`)
```
import * as cdk from '@aws-cdk/core';
import * as s3 from '@aws-cdk/aws-s3';

export class HelloCdkStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // new code
    new s3.Bucket(this, 'MyFirstBucket', {
      versioned: true
    });
  }
}
```

A couple of notes here
- The `scope`, `id`, and `props` variables are used as parameters for constructing both stacks and constructs. 
- **scope** - This is the parent of the stack. In this case, the parent is our app. The scope for the s3 bucket is the `HelloCdkStack` stack. 
- **id** - This is a way to uniquely identify resources across deployments. `MyFirstBucket` is the id of the s3 bucket construct. 
- **props** - These are values that will be used to define our construct. You can see in the `Bucket` constructor that we are passing an option of `versioned: true`. These are typically always going to be optional. 

### Synthesize App
> As I stated in my previous post, CDK uses AWS CloudFormation under the hood. We need to convert our app code into a CloudFormation template. You can take a look in the `cdk.out` directory to see what the template looks like!
```
cdk synth
```

### Deploy App
> The moment of truth! Let's deploy our bucket! If you have more than one stack defined you will need to choose one or use the `--all` option. You will see a nice progress bar.
```
cdk deploy
```

This is a [high-level](https://docs.aws.amazon.com/cdk/latest/guide/apps.html) diagram of the workflow of building and deploying a CDK app.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617147103874/5cQpq3W_6.png) 

### Modify App
> At some point, you will need to update your infrastructure. Let's update our S3 construct.
```
new s3.Bucket(this, 'MyFirstBucket', {
  versioned: true,
  removalPolicy: cdk.RemovalPolicy.DESTROY,
  autoDeleteObjects: true
});
```

> CDK offers a very nice tool to see what changes will be deployed. This is useful for see what resources will be removed, added, or modified. You can see what IAM policies will be adjusted.
```
cdk diff
```
> If you are happy with your changes you can deploy!
```
cdk deploy
```

### Destroy App Resources
> Finally, the day will come when you no longer need your resources! You can either just remove the resources you no longer need from your CDK code or destroy all the resources using a very useful command.
```
cdk destroy
```

## Writing Your Own Constructs
In addition to using the AWS Construct library, you can also create your own! Writing your own is as simple as composing a number of constructs already available via the Construct library. This allows you to encapsulate a number of constructs and easily reuse them. Let's look at a simple example:
```
export interface NotifyingBucketProps {
  prefix?: string;
}

export class NotifyingBucket extends Construct {
  constructor(scope: Construct, id: string, props: NotifyingBucketProps = {}) {
    super(scope, id);
    const bucket = new s3.Bucket(this, 'bucket');
    const topic = new sns.Topic(this, 'topic');
    bucket.addObjectCreatedNotification(new s3notify.SnsDestination(topic),
      { prefix: props.prefix });
  }
}
```
A couple of notes here:
- All you have to do to create your own Construct is use `extends Construct` and instantiate!
- This construct encapsulates an Amazon S3 bucket and a AWS SNS notification into a class named `NotifyingBucket`.
- You can use this custom construct by simply creating a new instance `new NotifyingBucket(this, 'MyNotifyingBucket');`

## Next Steps
I highly recommend giving the [CDK Workshop](https://cdkworkshop.com/) a try! They go through a bigger example with fine details. Read [Best practices for developing cloud applications with AWS CDK](https://aws.amazon.com/blogs/devops/best-practices-for-developing-cloud-applications-with-aws-cdk/).

## Conclusion 
The AWS CDK is a great tool for provisioning your cloud infrastructure! You can easily set up and destroy resources with a few lines of code. Keep this code in source control to monitor changes. Reuse your code for new projects or clients! I hope you enjoyed this tutorial! I will be writing some more in-depth articles going forward. 

## Special Thanks
I would like to take a moment to thank both [Nathan Peck](https://twitter.com/nathankpeck) and [Adam Keller](https://twitter.com/realadamjkeller) for helping me fine-tune this guide! They both gave me great feedback for improving this guide. 

Cheers!

🐦 Follow me on [Twitter](https://twitter.com/ninan_phillip) if you would like to see more content like this! 🐦

> Here is a link to my [Github Repo](https://github.com/fourgates/hello-cdk) with the fulling working example. 
