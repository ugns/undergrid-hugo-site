---
title: Rolling out a new mail server
featured_image: images/technology.jpg
type: post
author: jtbouse
date: "2012-09-21T12:00:59Z"
tags:
- amavis
- amavisd-new
- amazon
- apache
- aws
- clamav
- dkim
- dovecot
- ec2
- greylisting
- postfix
- PostfixAdmin
- rds
- roundcube
- sender policy framework
- ses
- spamassassin
- spf
- StartCom
categories:
- Projects
series:
- New Mail Server
---
So for the past few years I've been content to outsource my email services to [Web.com](https://web.com) with very few problems though lately I've had a few contacts report problems sending me email and I've ran into issues where they don't implement certain features I prefer to use (most notably user+extension email addressing). With that in mind I've set out to setup and re-implement my own mail server management and to 'eat my own dog food' as a consultant specializing in cloud service management I thought implementing it within [Amazon Web Services](https://aws.amazon.com). My experience with AWS has proven that I could make the migration and also save expenses which is never a bad thing.

For the scope of the project I was planning to utilize an [Elastic Compute Cloud (EC2)](https://aws.amazon.com/ec2/) instance that would run [Postfix](http://www.postfix.org/) and [Dovecot](https://www.dovecot.org/) IMAP/POP3 daemons to handle email sending & receiving. I also wanted to add [Apache2](http://httpd.apache.org/) with [Roundcube](https://roundcube.net/) for a webmail front-end. Of course this mail system needed to be secure and block spam that I have a love/hate relationship with (see [previous post]({{< ref "post/spam-be-gone/index.md" >}})) so I wanted to include [Amavis](http://www.ijs.si/software/amavisd/), [ClamAV](https://www.clamav.net/lang/en/) and [Spamassassin](https://spamassassin.apache.org/) integration. I couldn't forget the best weapon in the arsenal against spam, greylisting, either. With this scope I also figured it would need a database of some form so I would utilize Amazon's [Relational Database Service (RDS)](https://aws.amazon.com/rds/) to provide a MySQL instance. I'd also need a management interface for the mail system so I would make use of [PostfixAdmin](http://postfixadmin.sourceforge.net/) which is easy to tie in with both Postfix and Dovecot. Finally, all communication has to be secured so I would utilize X.509 secure certificates from my provider of choice, StartCom, as I have already taken care of all verification processes.

With that design scope in mind I've purchased the necessary AWS EC2 and RDS reserved instances and brought the EC2 instance online using the community Ubuntu Precise (12.04.1 LTS) AMI. I intend to deconstruct the steps taken to setup the mail infrastructure and document in multiple parts to give proper coverage to the configuration and the pitfalls I encountered with some of the content that I did find already available online.
