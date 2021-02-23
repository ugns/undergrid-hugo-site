---
title: Standardizing the Zone
featured_image: images/professional-solutions.jpg
type: post
author: jtbouse
date: 2021-02-23T01:27:57.000Z
publishdate: 2021-02-23T00:20:00.000Z
draft: false
summary: |
    Take a look as I break down how I approached the redesign of my
    AWS environment starting with how I approached standardizing all of my Route53 Hosted Zones.
    I wanted to maintain a base default expectation every time a new
    zone was created within the environment. Beyond just defining
    the domain for the zone, I wanted to insure that certain records
    and services were setup and/or established ready to be used.
tags:
    - aws
    - amazon
    - route53
    - mta-sts
    - ses
    - caa
    - s3
    - cloudwatch
categories:
    - Projects
slug: standardizing-zone
---

## Let's talk about zones

When starting out with a new domain you have certain expectations that Route53
handles automatically for you, like the NS and SOA records which come by default.
That is just the bare minimum to get you going though, you'll need a lot more
than that as you expect to the domain into use.

{{< highlight terraform >}}
    terraform {
        required_providers {
            aws = {
                source  = "hashicorp/aws"
                version = "~> 3.0"
            }
        }

        required_version = ">= 0.13"
    }

    variable "domain_name" {
        description = "Hosted Zone domain name"
        type        = string
    }

    variable "comment" {
        description = "Hosted Zone comment"
        type        = string
    }

    resource "aws_route53_zone" "this" {
        name          = var.domain_name
        comment       = var.comment
        force_destroy = false
        tags          = {}
    }
{{< /highlight >}}

#### How about an S3 bucket, or two?

When working within AWS it is generally a good idea to create an S3 bucket with
the same name as your domain. The big reason for this is that since S3 is a global
namespace *ANYONE* can create a bucket and the first one to do so gets it. Where
this becomes a problem is if you own the domain but not the S3 bucket, you can
then lose control of your website if you decide to host a static website from
S3. I have seen in the past where organizations setup an S3 static website for
their domain in a bucket that doesn't match their site hostname, maybe they put
it behind CloudFront, and things work until someone detects the bucket for that
hostname doesn't exist and decides to create the bucket and put up a fake site
that is suddenly now presented to viewers.

{{< highlight terraform >}}
    resource "aws_s3_bucket" "this" {
        bucket = aws_route53_zone.this.name

        website {
            index_document = "index.html"
        }

        tags = {}
    }
{{< /highlight >}}

So I find it is best to just go ahead and create a bucket for my zone, and
while I'm at it I also setup the traditional www.`domain` bucket as well and 
configure them both as static website buckets with the www.`domain` bucket set
to redirect to `domain`. While this prevents someone else from being able to take
the bucket name of your domain, it also gives you a great starting point as you
begin to setup your new domain's web presence as you could then just put a simple
"Coming Soon!" index.html page in the `domain` bucket.

{{< highlight terraform >}}
    resource "aws_s3_bucket" "www" {
        bucket = format("www.%s", aws_route53_zone.this.name)

        website {
            redirect_all_requests_to = aws_route53_zone.this.name
        }

        tags = {}
    }
{{< /highlight >}}

Though to get that page to be seen you have to have a DNS Resource Record created
that points to the S3 static website. This could be done when creating the zone
and S3 buckets initially but I refrain from doing so as more than likely I'll have
another deployment and I don't want competing deployments trying to change the record
on me. 

#### Are we going to use certificates?

Most current Certificate Authorities (CA) will now check for a [Certificate Authority 
Authorization][1] or CAA DNS record in your domain zone. This is simply a record that
states your public policy for whom can issue SSL/TLS certificates for your domain. Not
all CAs support CAA records but the number of those that do are increasing and include
most of the reputable ones. Amazon Certificate Manager (ACM) and Let's Encrypt both
support CAA lookups. With the increasing threat of malicious actors any extra level of
security is good so when setting up my zone I also include a CAA record limiting to
the CAs I will use, which invariably includes Amazon.

{{< highlight terraform >}}
    resource "aws_route53_record" "caa" {
        zone_id = aws_route53_zone.this.id
        name    = aws_route53_zone.this.name
        type    = "CAA"
        ttl     = 86400
        records = [
            "0 issue \";\"",
            "0 issue \"amazon.com\"",
            format("0 iodef \"mailto:hostmaster@%s\"", aws_route53_zone.this.name),
        ]

        lifecycle {
            create_before_destroy = false
        }
    }
{{< /highlight >}}

#### Sending email considerations

With a domain you are undoubtably going to want to be able to send email messages and
within Amazon the easiest way to do that is using [Simple Email Service][2]. Properly
setting this up now ensures it is ready to use when you need it, even if you have email
service being handled somewhere else this will ensure you can send message reliably out
of AWS. When I set this up for my zones I also include configuring it with DKIM authentication
and bounce & complaint notifications via an SNS topic along with the requisite DNS records.
In my Terraform deployment I've actually split this off into it's own module, there are
others out there on the [Terraform Registry][3] that you can evaluate though I consider
mine to be fairly complete for the task at hand.

First we request a domain identity and then domain DKIM generation for the domain. The
domain identity resource will then give us a verification token to prove ownership and 
we will receive 3 DKIM tokens to setup from the domain DKIM resource.

{{< highlight terraform >}}
    resource "aws_ses_domain_identity" "this" {
        domain = aws_route53_zone.this.name
    }

    resource "aws_ses_domain_dkim" "this" {
        domain = aws_route53_zone.this.name
    }
{{< /highlight >}}

Next we need to verify we control the domain with the verification token we received and
setup the 3 DKIM tokens in our new zone.

{{< highlight terraform >}}
    resource "aws_route53_record" "amazonses" {
        allow_overwrite = true
        zone_id         = aws_route53_zone.this.id
        name            = "_amazonses"
        type            = "TXT"
        ttl             = 1800
        records         = [aws_ses_domain_identity.this.verification_token]

        lifecycle {
            create_before_destroy = false
        }
    }

    resource "aws_ses_domain_identity_verification" "this" {
        domain = aws_route53_zone.this.name

        depends_on = [aws_route53_record.amazonses]
    }

    resource "aws_route53_record" "dkim_token" {
        count = 3

        allow_overwrite = true
        zone_id         = aws_route53_zone.this.id
        name            = "${element(aws_ses_domain_dkim.this.dkim_tokens, count.index)}._domainkey"
        type            = "CNAME"
        ttl             = 600
        records         = ["${element(aws_ses_domain_dkim.this.dkim_tokens, count.index)}.dkim.amazonses.com"]

        lifecycle {
            create_before_destroy = false
        }
    }
{{< /highlight >}}

The next piece required to finish setting up SES for our domain is to setup the
DNS records that Amazon expects to find. We will request SES use the MAIL FROM
to be used as bounce.`domain` and Amazon will require us to setup an MX record
to receive those bounce messages and an SPF TXT record for this MX. As this is
only for receiving bounce messages the SPF record does not need to be modified.

{{< highlight terraform >}}
    data "aws_region" "this" {}

    resource "aws_ses_domain_mail_from" "this" {
        domain           = aws_ses_domain_identity.this.domain
        mail_from_domain = format("bounce.%s", aws_ses_domain_identity.this.domain)
    }

    resource "aws_route53_record" "ses_mail_from_mx" {
        allow_overwrite = true
        zone_id         = aws_route53_zone.this.id
        name            = aws_ses_domain_mail_from.this.mail_from_domain
        type            = "MX"
        ttl             = 600
        records         = [format("10 feedback-smtp.%s.amazonses.com", data.aws_region.this.name)]

        lifecycle {
            create_before_destroy = false
        }
    }

    resource "aws_route53_record" "ses_mail_from_txt" {
        allow_overwrite = true
        zone_id         = aws_route53_zone.this.id
        name            = aws_ses_domain_mail_from.this.mail_from_domain
        type            = "TXT"
        ttl             = 600
        records         = ["v=spf1 include:amazonses.com -all"]

        lifecycle {
            create_before_destroy = false
        }
    }
{{< /highlight >}}

You may note that I included the `allow_overwrite` setting to true which tells
Terraform to overwrite these records in Route53 if they exist. As this should
authoritative for these records and nothing else should be setting them this
should be okay.

While optional, it is a good idea to be sure you stay aware of any complaints
or bounces received by Amazon when using SES as these can limit your ability to
use the service if the rate becomes too high. You can additionally setup [CloudWatch][4]
alerts on an account level as these metrics are not per zone. If you have an
existing SNS topic you can use that rather than creating a new one. I leave it
with only Bounce and Complaint notification types, there is a third Delivery
which you could enable while getting started but once active and if it sends any
amount of email could be unnecessary noise so I don't enable it.

{{< highlight terraform >}}
    resource "aws_sns_topic" "sns" {
        name_prefix  = format("%s-", replace(aws_route53_zone.this.name, ".", "-"))
        display_name = format("%s CloudWatch Alarms", aws_route53_zone.this.name)

        tags = {}
    }

    resource "aws_ses_identity_notification_topic" "this" {
        for_each = ["Bounce", "Complaint"]

        topic_arn                = aws_sns_topic.sns
        notification_type        = each.value
        identity                 = aws_ses_domain_identity.this.domain
        include_original_headers = true
    }
{{< /highlight >}}

If you didn't use an existing SNS topic and created a new one as the template above
demonstrates then you will need to create a subscription to receive the notifications.
Unfortunately due to the verification steps to accept the subscription Terraform
is unable to handle `email` or `email-json` subscriptions to a topic though if you
had a SQS, Lambda function or an HTTP/HTTPS application that received your alerts
you could add an `aws_sns_topic_subscription` resource.

With this in place we have a Route53 Hosted Zone baseline to begin working from.
On top of this I would then add my MTA-STS and DMARC settings which I will go over
in another post so check back again for that update. With this solution I wrap it
all up in a git repository and create a [Terraform Cloud][5] workspace for each of
my zones with version control workflow. Operating it in this way, if I make a change
to my zone template it is applied to all of my zones uniformly which afterall is the
whole goal of this exercise.

[1]: https://en.wikipedia.org/wiki/DNS_Certification_Authority_Authorization "CAA DNS record"
[2]: https://aws.amazon.com/ses/ "AWS SES"
[3]: https://registry.terraform.io/ "Hashicorp Terraform Registry"
[4]: https://docs.aws.amazon.com/ses/latest/DeveloperGuide/event-publishing-retrieving-cloudwatch.html#event-publishing-retrieving-cloudwatch-metrics "AWS CloudWatch SES metrics"
[5]: https://app.terraform.io/ "Hashicorp Terraform Cloud"
