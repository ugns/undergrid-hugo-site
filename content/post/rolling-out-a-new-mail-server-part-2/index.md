---
title: Rolling out a new mail server - part 2
featured_image: images/technology.jpg
type: post
author: jtbouse
date: "2012-09-21T23:33:23Z"
slug: rolling-out-a-new-mail-server-part-2
tags:
- amazon
- aws
- ec2
- rds
- route53
- ses
categories:
- Projects
series:
- New Mail Server
---
If you're not going to try running this under AWS then you can pretty much skip on ahead to the rest of the configuration.

So the obvious place to start is in setting up the EC2 instance. If you just want to test this out a t1.micro instance is plenty big enough and the on-demand pricing to run it for a few days won't cost a lot. In fact if you just signed up for Amazon Web Services you could run a t1.micro free for a year with their free tier service. As my expected load is not more than the t1.micro can handle I actually got a t1.micro EC2 reserved instance and it runs me less than 7.50 USD/month give or take depending on bandwidth and storage costs. If you want/need some AWS consultancy get in touch as I've got plenty of experience with AWS and saving my clients plenty in recurring costs.

As I have an Ubuntu Precise (12.04.1 LTS) system already I installed the [cloud-utils](https://packages.ubuntu.com/cloud-utils) package and used the ubuntu-cloudimg-query tool to find the current AMI. For a t1.mirco instance you need an EBS backed AMI so you can find the current using:

{{< highlight bash >}}
  $ /usr/bin/ubuntu-cloudimg-query precise amd64 ebs us-east-1
{{< /highlight >}}

With the AMI value you can then fire off the EC2 instance using either the CLI tools if you have them installed or the [AWS console](https://console.aws.amazon.com/ec2/home/). Be sure when you start it up that you specify the SSH key name or you won't be able to log into the instance once it's started.  With the Ubuntu AMI you'll not be able to login directly as **root** so you'll have to log in as **ubuntu** and then `sudo -i` to gain root access. You'll want to setup a security group with the proper inbound ports opened up. For now you'll need at least the SSH port open to access the EC2 instance. For testing you can restrict to your own IP but for regular operations you'll need to allow from any IP address (0.0.0.0/0). You'll need to allow SMTP (25/tcp) and I include 587/tcp for the mail submission port. You'll also need to allow IMAP (143/tcp) & POP3 (110/tcp), you can also IMAPS (993/tcp) & POP3S (995/tcp) for TLS even though the other ports will support STARTTLS.

Once you have the EC2 instance you'll also need the RDS instance up and running. Again I went with a t1.micro instance as it provided plenty of power for what I was planning to use and with Amazon's ability to easily migrate to larger instance sizes when needed why spend the extra until I had to. When starting the instance I find it easier to just use **root** as my admin user as I've ran into cases where package installs won't use a non-root admin user. I didn't have a default database installed with the instance and gave it the minimum 5GB volume size. In a high availability situation you would of course want to use a multi-AZ RDS instance but for this a standard RDS instance will suffice. You'll need a RDS Security Group setup as well that allows access from your EC2 security group. This will allow the EC2 instance to communicate with the RDS instance. You could also include your IP if you want to access it directly though I did all my changes from the EC2 instance directly.

The last Amazon service we have yet to setup is the [Simple Email Service (SES)](https://aws.amazon.com/ses/) that I use for sending email from my domains out.If you're using [Route53](https://aws.amazon.com/route53/) this is very quick as SES provides the ability to automatically add the DNS entries for domain verification and to enable DKIM signing. The other perk is that sending through SES from EC2 is free. I went ahead and verified my domains that I'm going to be configuring the mail server to handle along with those external addresses I have setup as alternative identities like my Gmail and my Debian developer addresses. While the configuration I put in place will send with those addresses directly if I don't verify them I want to ensure they get sent out via SES.

The final pieces to the Amazon side of things for now is that I assigned an Elastic IP (EIP) to the EC2 instance to use later and as I use Amazon's Route53 for the DNS zone for my services in the cloud I setup a CNAME entry in the zone for my domain that points to the RDS instance hostname that I was given when the RDS instance came online.

With all of this in place we're ready to move on to the installation and configuration of the various software packages which we'll start to go over in the next posting.
