---
title: Starting Over Again but Not Completely
type: post
author: jtbouse
date: 2021-02-19T05:27:29.000Z
publishdate: 2021-02-20T00:00:00.000Z
summary: |
    Eventually change comes to us all. We learn new skills and find flaws in 
    existing tools we have been using to work with. That time is upon us here...
    After coming up against challenge after challenge with using CloudFormation 
    I have since turned to Terraform and begun finally completing the task
    of automating the entire environment and minimizing manual efforts.
tags:
    - aws
    - amazon
    - terraform
    - infrastructure as code
categories:
    - Projects
slug: starting-completely
---

## Get the lay of the Land

What is it they say about change? Oh right! The only thing consistent is change.

We've all worked in those environments that are constantly changing. I've referred
to some in the past as suffering from "shiny tool syndrome" in fact. Something new
comes out and you suddenly have a new project to switch everything over to use it.
I have generally tried to keep from changing things up without having a really good
reason to, just because I like the consistency in having things that already work...
well continue to work.

Like many organizations starting the cloud migration, things move quick and fast. A
lot of times things are done by hand through the console as you're still learning and
making changes quickly. Eventually you make the move to standardizing things, in AWS
you typically see a move towards CloudFormation templates to do this. I had previously
done a lot of my EC2 instance bootstrapping by tying it back to Puppet and eventually
SaltStack. This worked good but then I also figured out how I could a lot of the initial
heavy lifting still within the metadata through CloudFormation do to installations and
leave Puppet or SaltStack to handle the actual configuration. The infrastructure itself,
well there in lies the rub. Some of it was templatized, but sometimes that was after the
fact and CloudFormation doesn't handle incorprating existing resources easily. Now I know
some will say it does support importing drift, but that first assumes you have a stack
template already and the environment has drifted from it by manual revisions. The other
side effect is that removing the CloudFormation stack also removes the resources, which
let's face it is another reason why you want to have it templatized and standardized in
first place.

Enter Terraform... One of the key features that I found appealing in Terraform was the
ability to import existing resources and take over managment of them. This was a very
keen missing feature in CloudFormation. The biggest drawback against it was that it was
a third-party solution while CloudFormation was obviously more authoritative when working
within AWS afterall. I had also already started down the road investing time and effort
to render my environment into CloudFormation templates by this point, but I kept running
into a similar issue time and time again. To put it bluntly, support for new features.

As new features were released within AWS I inevitably found use cases for them but at the
same rate I found CloudFormation wasn't up to the task. I got very used to adding the
phrase "but not with CloudFormation" every time I would read a new feature being supported
and it stated that it was supported using the AWS SDK, CDK or CLI. This was because support
for CloudFormation always came much later, sometimes many months later. Of course, AWS
Support was always keen to point out the option of using Custom Resources in the form of
Lambda functions which used the SDK to perform the same thing you expected CloudFormation
to be able to perform. This turns out to be a boon for Terraform in the "Pros" column as it
uses the SDK underneath so support tended to appear a lot sooner than in CloudFormation.

So now you see where things have lead me, now let's discuss where I want to see them go!

## Lay out the objective

So given the advantages of Terraform, I'm moving away from CloudFormation to define my
Infrastructure-as-Code format. As I'm looking to replace the CloudFormation I already have
written, I might as well go ahead and ensure that even the resources I don't have deployed
with CloudFormation get added to the Terraform deployment.

With the format to write my IaC deployments in and we have a scope of what to include, we
can expand on this by stating a goal as being able to rebuild everything with minimal manual
effort involved. The biggest incumberance to that is dynamically generated content from
a database. So lete's make the move to use a static site generator and we can retire the
old RDS database which always tends to be a large portion of the monthly bill.

Ultimately, we also want everything deployed following some standardization and best practices.
With multiple domains to manage this will give plenty of test cases for everything built which
should help in designing robust but accomoodating deployment logic. It also means we can
follow a convention over configuration methodology to keep things simple.

A lot of this effort will also mean moving more towards a more serverless design which hopefully
should lead toward lower costs allowing more new things to come while staying within budget,
which given our pandemic driven days is always a plus.

So I hope you'll stick with me as I begin to work on the tasks at hand. The obvious first choice
to start with will be in domain handling as everything depends on that and sadly I have to admit
is one of those areas that I never got to implementing any automated deployment for so it will
be a great greenfield project to start with.

For my next post on the subject we'll take a look at how I will be approaching setting up all
of my Route53 Hosted Zones using Terraform. After we have our domains standardized under Route53
we can begin to expand from there setting up DNSSEC, SSL/TLS certificates using Certificate
Manager and securing e-Mail communications.
