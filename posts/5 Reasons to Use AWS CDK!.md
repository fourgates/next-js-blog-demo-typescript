---
title: "5 Reasons to Use AWS CDK!"
date: "2020-01-01"
---

## What is AWS CDK?
The [AWS Cloud Development Kit (AWS CDK)](https://aws.amazon.com/cdk/) is an open-source framework to help you provision cloud application resources by using your favorite programming languages! You can choose from TypeScript, JavaScript, Python, Java, or C#. 

Simply put, you can leverage the AWS CDK to write code that will provision (create) AWS resources. If you are familiar with AWS CloudFormation, you should feel pretty comfortable using the AWS CDK. The CDK actually uses CloudFormation under the hood!

## Why should you use the CDK?
Provisioning cloud applications can be very challenging. This may require a lot of manual work, maintaining templates, or using domain-specific languages. The AWS CDK allows you to harness the power of languages you already know to model your applications. You can also leverage **high-level constructs** that do a lot of the heavy lifting for you!

Here are MY favorite reasons:
1. You can spin up cloud infrastructure using **TypeScript**!
2. It allows you to **reuse code**. This is extremely useful when setting up multiple environments or applications.
3. The AWS CDK framework was built with **best practices** in mind. Using their framework and high-level constructs helps ensure, but does not guarantee, that you are using them correctly.
4. You can commit your code and leverage the AWS CDK as **Infrastructure as Code (IaC)**. This allows each change to be documented in source control and gives you the ability to utilize the same systems as your application source code. 
5. You need to **write less to achieve more** when compared to CloudFormation or Terraform. 

🐦 Follow me on [Twitter](https://twitter.com/ninan_phillip) if you would like to see more content like this! 🐦
## What are Constructs?
Constructs are a fundamental building block of AWS CDK. Each construct represents a "cloud component". A construct encapsulates everything AWS CloudFormation needs to create the component. 

A construct can be as simple as an Amazon Simple Storage Service (AWS S3) bucket:
```
    new s3.Bucket(this, 'MyFirstBucket', {
      versioned: true
    });
```

It can also be abstract enough to create higher-level components that require multiple AWS resources. A great example of this is AWS Elastic Container Service (AWS ECS) constructs. This can require many components such as a cluster, service, task, and an Elastic Load Balancer (ELB). You can configure this with a handful of lines of code:

```
    // Create a load-balanced Fargate service and make it public
    new ecs_patterns.ApplicationLoadBalancedFargateService(this, "MyFargateService", {
      cluster: cluster, // Required
      cpu: 512, // Default is 256
      desiredCount: 6, // Default is 1
      taskImageOptions: { image: ecs.ContainerImage.fromRegistry("amazon/amazon-ecs-sample") },
      memoryLimitMiB: 2048, // Default is 512
      publicLoadBalancer: true // Default is false
    });
```

The AWS CDK provisions these resources in a safe and repeatable manner. You can reuse this code to provision multiple environments. You can also compose your own custom constructs allowing you to start new projects much faster!

## Conclusion
At a high level, that's it! AWS CDK is a useful library that allows you to use your favorite programming language to provision cloud infrastructure. It allows you to quickly generate AWS resources, promotes best practices, and allows you to reuse code! 

> Stay Tuned -- I am writing several tutorials demonstrating how to use the AWS CDK!

🐦 Follow me on [Twitter](https://twitter.com/ninan_phillip) if you would like to see more content like this! 🐦
## Resources
- [AWS CDK Construct Library](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html)
- [AWS CDK TypeScript Docs](thttps://docs.aws.amazon.com/cdk/latest/guide/work-with-cdk-typescript.html)
- [AWS ECS Example](https://docs.aws.amazon.com/cdk/latest/guide/ecs_example.html)
- [AWS CDK Workshop](https://cdkworkshop.com/)
- [AWS CDK Workshop](https://cdkworkshop.com/20-typescript.html)
- [Github Examples provided by AWS](https://github.com/aws-samples/aws-cdk-examples/blob/master/typescript/ecs/fargate-service-with-local-image/index.ts)
