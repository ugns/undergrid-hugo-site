---
title: What about container security?
type: post
author: jtbouse
date: "2018-08-27T13:47:39Z"
tags:
- amazon
- Anchore
- aws
- cloudformation
- devops
- ecs
- PostgreSQL
- rds
- software
categories:
- Projects
---
With normal legacy server security you would routinely scan your servers for vulnerabilities, but with the move to using containers this methodology for vulnerability detection doesn't exactly fit as you would typically build the container and move it along through your environments during the software development life cycle. So how do we check our containers to ensure they are as secure as our old servers? How do you know your image is still secure after it's been built?

This was a problem that I had to address when making the move in my own environment to use a more container-based solution. While being a big proponent for open-source software, but still needing to look to work within corporate environments I had to find the solution that would work for both. Thankfully the great team over at [Anchore](https://anchore.com/) have worked to build just that. Anchore has built a strong [open source engine](https://anchore.com/opensource/) capable of doing the vulnerability scans I needed for my Docker containers, but still have the [Emterprise support offerings](https://anchore.com/enterprise/) to make corporate leadership happy.

While Anchore had a [provided solution](https://github.com/anchore/anchore_deployment) to deploy cleanly into a Kubernetes environment, that didn't exactly work for me within Amazon Web Services using Elastic Container Service. I therfore had had to work my way through the caveats of AWS to get a full solution working. The process took many iterations to get to the point I have it working today, so let's take a look at some of the aspects I had to work through to get it to work.

While designing my solution I wanted to be able to provide auto-scaling of the services that make up Anchore Engine. This was made easier in the v0.2.x version which removed the need for setting a Host ID and instead allowed to use a UUID instead. The trick however was when looking at ECS and placing Anchore Engine behind an ALB. Each ALB Listener can only listen on one port and each Anchore Engine service listened on it's own unique port. You then add to that the fact that an ECS Service can only attach a container to exactly 1 ALB Target Group. This meant that each of the Anchore Engine services would need to be deployed separately, so instead of 1 ECS Task Definition and 1 ECS Service I would need to deploy 6 of each. These Anchore Engine services were the API (anchore-api), Kubernetes webhook (anchore-kubernetes-webhook), Policy Engine (anchore-policy-engine), Simple Queue (anchore-simplequeue), Catalog (anchore-catalog) and the Worker (anchore-analyzer). All except the worker will be placed behind the ALB so they would be assigned to their own Target Groups and Listener. The Worker talks with the other services through the ALB and doesn't need to be directly accessible.

{{< gist jbouse d5e80b3980e3e35def350596b244a532 "catalog-service.yaml" >}}

Above you can see an excerpt from the ecs/anchore.yaml template demonstrating all the resources for the Catalog service. This is a representative example of how each of the other services (API, Simple Queue, Policy Engine and Kubernetes webhook) would be setup with one exception. Only the Catalog requires the _TaskRoleArn_ property be set if you're going to make use of the Elastic Container Registry (ECR). If you're not going to use ECR then you could leave the _TaskRoleArn_ property off completely. The rest of the services all can run without any specific IAM permissions being granted to them.

{{< gist jbouse d5e80b3980e3e35def350596b244a532 "worker-service.yaml" >}}

The Worker analyzer as mentioned only requires the ECS TaskDefinition and Service resources to be defined and since it will also need access to ECR it has the _TaskRoleArn_ set like the Catalog Service did. You will also note that in both excerpts I'm defining several environment variables to be made available in the TaskDefinition for the services. These are all the same regardless of the service they are listed for with the exception of the _ANCHORE\_ENGINE\_SERVICES_ variable. All of the environment variables except the _ANCHORE\_ENGINE\_SERVICES_ are used within the config.yaml that every service references through the bind volume mount.

{{< gist jbouse d5e80b3980e3e35def350596b244a532 "config.yaml" >}}

Taking a look at the config.yaml file that I have my ECS cluster instances download from an S3 bucket you can see where each of the environment variables are used. You will also note the absence of the _ANCHORE\_ENGINE\_SERVICES_ variable being used in the config file. This absence is not an oversight but rather that the Anchore Engine services themselves use this variable to determine which services to start up. Within the config file all of them have _enabled_ set to true, but the _ANCHORE\_ENGINE\_SERVICES_ environment variable overrides this and only starts the service listed in the variable. As we're starting one service per container, this variable will only hold the name of that service.

The only other requirement for Anchore Engine is the need for a PostgreSQL database server. Within my environment I made use of an RDS PostgreSQL instance that I already had running. Unfortunately I'd stood it up manually without a CloudFormation template so I created a data/rds-legacy-postgres.yaml template that I provided the necessary values to be exported. I intend to move this PostgreSQL instance from the old Default VPC into my custom VPC and deploy it via CloudFormation, but that has not been done yet. By creating this legacy template I've made it easier on myself to migrate to this new RDS instance when I do. I also added some extra functionality into my ecs/anchore.yaml template that I will cover in another post, which includes the Enterprise UI and it's requirement for a Redis cluster. While this covered the corporate needs I mentioned it wasn't required to use the open-source Anchore Engine itself so I've left it to be its own post and outside the scope of this one.
