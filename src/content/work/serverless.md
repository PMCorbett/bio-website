---
title: Musings on Serverless
publishDate: 2023-03-14 00:00:00
img: /assets/serverless-logo.png
img_alt: Serverless logo
description: |
  Thoughts on my experience with using the serverless framework
tags:
  - Serverless
  - Lambda
  - AWS
---

## Musings on Serverless

I love Serverless. It is brilliant.

## More detailed musings on Serverless

Serverless is a fantastic tool for orchestrating cloud infrastructure in a way that means you don't have to worry too much about the said infrastructure. The plugins available can make life extremely simple to add functionality without having to know any of the difficult information about cloud.

### How I've used Serverless

I've exclusively used Serverless on AWS. I have (as much as possible) run my serverless deploys as part of a CI/CD pipeline. I've mostly written client apps that are statically deployed in S3, and have Lambda functions that they call. I've also written serveral Lambda functions that do things based on SNS notifications, or other event trigger.

### My approach to Serverless

In general my approach to programming is to always assume that someone cleverer than me has already solved the problem I am trying to solve, and so I look around the internet in the hope that they have released their work for free. The Serverless plugins are excellent and numerous, and they seem to have been written by people cleverer than me. So I've used them extensively.

### Cool plugins

#### fullstack-serverless

This is a nice plugin that pops your lambda functions behind their api gateway and then provides a cloudfront distribution in front of them and a client app in a S3 bucket. Very neat and tidy for having your client and lambda code in the same codebase and deploying in the same stack.

#### serverless-offline

Allows for offline development, an absolutely crucial plugin.

#### serverless-print-dots

This seems like a really silly plugin, it just prints a dot ever few seconds when there is no other output from your Serverless deploy. However, when doing CloudFront updates from CI, I had a problem with CircleCI killing the task when there was no output. And this plugin saved the day.

#### serverless-domain-manager

Easily manage domains in SLS.

### Tricky bits

#### Yaml

At some point in a projects life, the serverless config file will get to be too big. After a certain point, yml files become unreadable. This isn't a problem with serverless, I just need a place to vent.

#### CloudFormation

You often have to write a little bit of cloudformation in the serverless config file. This is fine mostly, but I do find it quite cumbersome. First you write a bit of cloudformation to create an SNS channel in the resources section, then you go to the provider->iamRoleStatements section and write out what privileges you need for the SNS channel, then you right a reference to that channel and poke it into the environment of the lambda function. And by this time, you've scrolled up and down 400 times, got lost twice, and messed up the indentation four times. Then you get it all sorted, try to deploy it and realise you've missed off the region.

Perhaps none of that is really CloudFormation's fault, and it is in fact mine 🤷‍♂️.

### Lovely bits

The general ease of not having to worry about docker files and ecs clusters and provisioning this, scaling up that and all of that jazz is a massive weight off my mind. (And going back in time further - deploying with capistrano or terraform, or just ftping files onto a server).

The way that lambda functions sit in the whole of cloud architecture is rather nice. Being able to have an SNS message, DynamoDB event, http event, SQS, Transcribe event, or even another lambda function directly triggering a lambda function is SOOOOOOO nice. It means you can tack a lambda function onto the end of another process super easily. Or you can build a complicated event model, and it all just hangs together with an elegance that soothes my soul.

Websockets are so simple with Serverless and AWS Websocket API. Setting up a websocket is as easy as writing `- websocket` in the event list. And sending data to the socket is just `.postToConnection()`, so easy, so fast, so cool.
