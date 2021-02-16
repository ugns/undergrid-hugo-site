---
title: Working with ECS
featured_image: images/professional-solutions.jpg
type: post
author: jtbouse
date: "2018-08-22T01:18:40Z"
tags:
- amazon
- aws
- cloudformation
- ecr
- ecs
- efs
- kms
- vpc
categories:
- Projects
---
By this point we should have a VPC in place and our Private tier subnets can route out either a NAT Gateway or VPN Connection that performs NAT, we can then begin to look at deploying applications into the environment in earnest. In my environment I've made the choice to use containers as much as possible so the most logical next step is to setup an [Elastic Container Service (ECS)](https://aws.amazon.com/ecs/) cluster. Once we have our cluster up and running we will then be able to deploy applications into it. ECS is Amazon's Docker implementation and operates mostly as any Docker cluster would with a few exceptions, most notably is the absence of Swarm usage. Instead of using Swarm, Amazon has their ECS Agent that runs as a container itself that communicates with the AWS API and Docker to pull images and launch containers as necessary. It also has maintenance cleanup processes that run on a schedule.

{{< gist jbouse 2c797e4f28bf34f35776de9b2d5fae47 "ecs-cluster.yaml" >}}

Taking a look at my ecs/cluster.yaml template you'll see most of the parameters are optional and those that are required have fairly sane default options to stand up a a small cluster. The only real parameter you have to provide is the name of your VPC stack so that it can deploy within it. Some of the other parameters of note would be the **_LoadBalancerScheme_** which could be set to internal if you wanted to have a cluster that was only accessible within your VPC and didn't need Public access. The ALB is utilized to check the ECS instance health via the default action checking that the ECS Agent is responding. It also sets up a HTTP and an optional HTTPS listener that are exported and able to be extended by other application stacks as we'll see later. You can specify the ARN of a certificate to be used for the HTTPS listener. This could be imported, but due to a limitation in CloudFormation that only allows the use of email verification of certificates registered through the [AWS Certificate Manager (ACM)](https://aws.amazon.com/certificate-manager/) instead of using Route53 domain verification I still prefer to set them up manually and reference the ARN when needed.

The **_DockerAuthType_** and **_DockerAuthData_** parameters allow you to pass private registry credentials to the ECS Agent to be able to pull down private repositories. This is not necessary if you're only using the [Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/) as the permissions for that are granted via the IAM Instance Profile the ECS instance runs under.  The DockerAuthType can be either &#8216;docker' or &#8216;dockercfg' and the DockerAuthData then needs to contain the appropriate JSON string. You can read more about this in Amazon's [Private Registry Authentication for Container Instances](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/private-auth-container-instances.html) page. While they have sense added [Private Registry Authentication for Tasks](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/private-auth.html) which allow for providing these credentials via AWS Secrets Manager recently, I have not yet seen CloudFormation updated to accept the repositoryCredentials property at this time. Once that is available via CloudFormation I will most likely update my template to use that method instead as it would allow for storing the credentials encrypted.

It's also worth noting that the template has the ability to launch the cluster using EC2 Spot instances which can save you money and allow you to be able to run larger instance types. The best advice here is to set the _**SpotPrice**_ to the normal on-demand price for your desired instance price and you'll be assured that an instance will be able to run but you'll get the cheapest price available for it.

The Cluster Scaling Parameters with with the Auto Scaling Group's Lifecycle Hook which allows the instance to drain any running containers to another cluster instance before being terminated. This allows for graceful migration of container services when scaling the cluster down. The maximum CPU and Memory should be set to the normal container reservation values so that the Lambda function can more accurately calculate how many containers could still be scheduled to run on the cluster. The Shortage and Excess thresholds are then used to determine if the cluster should scale up or down based on whether the available number of containers that could still be executed is either below or above those thresholds. Setting these values can take  some trial and error based on your own cluster workload so you can monitor the value through the custom CloudWatch metric that holds the value.

Much of the actual bootstrap magic is done within the LaunchConfiguration Metadata that starts at line 481. The steps are broken down into separate logical config sets that are executed by the cfn-init helper script upon boot and again when triggered by the cfn-hup process when updates are made to the stack. The _efsmount_ config set is shown to be conditional on line 489 depending on the HasEFSVolume condition that checks to see if the EFS Stack name has been provided as a parameter. If this is provided it will mount the EFS volume to the EFSDirectory that defaults to /mnt/efs on each cluster instance. This allows your containers to have a common volume location to use regardless of which instance in the cluster it is executed on.

{{< gist jbouse 2c797e4f28bf34f35776de9b2d5fae47 "efs-volume.yaml" >}}

Looking closer at the EFS Volume itself you see that the efs/volume.yaml template requires very few parameters to get setup. As with the ECS Cluster template you need to specify the VPC Stack name and as we had to with the VPN Connection template we have to tell the stack how many AZs we have within our VPC so that it can setup the proper number of EFS Mount Targets. The other parameters can be left as default; however, if you'd like to have the data on the volume encrypted at rest then you may which to enable encryption and enable having the key automatically rotated annually. When encryption is enabled it will create a Customer Master Key (CMK) that is used to encrypt only this EFS volume and optionally rotate the key annually if enabled.

Once the stack is created it will allocate the mount targets in each AZ and export both the SecurityGroup and the FileSystem that other stacks like the ECS Cluster will require. The security group is setup so that resources which are assigned it are allowed to mount the volume over NFSv4 (2049/TCP). The rules of this security group should never need to be modified by any other stack, it just needs to be available to assign to the resource as we do in the ECS template on line 810.

Obviously the EFS Volume stack would need to be created before it can be provided to the ECS Cluster stack, though a nice feature of CloudFormation is that you can actually launch the ECS Cluster stack without EFS included and then update the stack later to add EFS. In fact once the stack is created you can update the stack using the existing template without having to re-upload the template. This is particularly handy when the ECS Optimized AMI is updated by Amazon as they do regularly. Since the ECS Cluster template references the AMI using the public SSM Parameter Store name it is looked up whenever the stack is updated and returns the appropriate AMI ID for the given region the stack is being executed within.

I do hope this was informative and forms a starting point for your own work. Feel free to ask questions about either template. In my next post I'll move on and begin to deploy some of the applications used within my VPC continuing to build on the topography we've setup to this point.
