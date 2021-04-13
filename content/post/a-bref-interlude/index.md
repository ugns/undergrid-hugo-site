---
title: A Bref Interlude
type: post
author: jtbouse
date: 2021-02-27T00:10:25.000Z
publishdate: 2021-02-26T00:00:00.000Z
draft: false
summary: |
    I'll take a brief break from the direct AWS infrastructure and discuss
    another piece of the puzzle I needed to address while moving all of my
    website to operate within a more serverless capacity.
    I had my GNU Privacy Guard Key Policy site I'd developed back in '08
    using PHP3 and until recently had not really found a way to move it
    to the cloud without running instances.
tags:
    - aws
    - amazon
    - lambda
    - serverless
    - terraform
    - bref
    - symfony
    - php
    - gpg
    - key policy
    - pgp
    - software
categories:
    - Projects
slug: bref-interlude
---

## Revisiting an old friend

Previously in my [GnuPG Key Policy Manager]({{< ref "post/gnupg-key-policy-manager/index.md" >}})
post I had briefly discussed my site that I published my key policy relating to GPG/PGP key usage
but never really talked about what I had put behind it to make it work.

I had originally written it using the then current PHP 3.x and this was really before the popularity
of single page applications (SPA) that we see so common today. So I had written the entire site with
it's own "routing" using the PATH_INFO from the server. It was very brittle and it was no more so
apparent then when it came time for subsequent PHP upgrades. Every time my server would have PHP
upgraded, I would find that it broke and I had to go back in and modify it just enough to get it
working but not change it's functionality. This went on until I'd finally last updated it to run
cleanly running on PHP 7.x and now with 8.0 coming out and wanting to remove the usage of instances,
or even containers for that matter, unless absolutely necessary I needed to take a good hard look at
my venerable old friend.

## Challenge Accepted

So looking to upgrade and re-implement my key policy site I looked at various frameworks that were
now around to find one that would work and easy enough for me to pick it up quickly as I had never
used it before. [Vue.js][1] and [Angular][2] were both looked at but seemed to be a bit more than
what I needed for this and I felt the learning curve would be a bit more involved. I did keep them
on my radar though as I always have other project ideas so I am very likely to go back and take
another look atone or both of them for something here in the future.

Next I came across [Symfony][3] and [Laravel][4]. Both seemed to be promising for several reasons.
First of which was that they were less JavaScript but provided the MVC templating and routing I
needed. They were also both supported by [Bref][5] which I had already identified to help with the
other piece of the puzzle. I had already played around a little with Bref to test running PHP code
in Lambda so felt comfortable with working with it. By default Bref supported Serverless and SAM AWS
CloudFormation templates, not exactly what I was looking for as I was moving things towards
Terraform but I wouldn't let that stop me from getting the project underway.

## The Path Forward

I was mildly suprised at how easy it was for me to essentially re-write the entire key policy site
from the single PHP page I'd written over a decade ago using Symfony. I started by breaking down the
pieces of the page display and actually write MVC templates using Twig. I had sort of done this with
functions within my PHP code, but I had the PHP code logic intersparsed with the HTML code it would
output. I knew I could do better than that and if I was going to be re-writing the project I might
as well do it. I was even able to implement using flash messages to handle when recieivng a URL with
a checksum to validate. I also was able to better implement checksum validation to be both backwards
compatible and future proof. 

{{< highlight php >}}
    protected function validate_checksum(int $policyId, string $checksum, string $algo): void
    {
        $policy = $this->get_policy($policyId);
        if (hash_file($algo, $policy->getRealPath()) == $checksum) {
            $this->addFlash(
                'success',
                strtoupper($algo) . ' Checksum valid'
            );    
        } else {
            $this->addFlash(
                'warning',
                strtoupper($algo) . ' Checksum NOT valid'
            );
        }
    }
{{< /highlight >}}

I had initially started using MD5 checksums and then briefly had used SHA1 before moving to SHA256
which I use currently. The new system is totally ready to handle any other checksum formats in the
future with minor changes.

{{< highlight php >}}
    /**
     * @Route("/policy/{policyId<[0-9]{8}>}/{checksum<[0-9A-Fa-f]{32}>}", name="policy_md5sum")
     */
    public function md5sum(int $policyId, string $checksum): Response
    {
        $this->validate_checksum($policyId, $checksum, 'md5');

        return $this->redirectToRoute('gpg_policy_show', ['policyId' => $policyId]);
    }
{{< /highlight >}}

Working with the Symfony annotations I found I could really define the URI patterns while providing
the reference linking and validation. I also really loved that my code was now clear inside the PHP
code and then the Twig template partials handled the display for me.

{{< highlight twig >}}
    {% for label, messages in app.flashes(['success', 'warning']) -%}
        {% for message in messages %}
            <div class="chksum {{ label }}">
                {{ message }}
            </div>
        {% endfor %}
    {% endfor -%}
{{< /highlight >}}

## Future improvements

While I got the site redesigned with pretty much a complete re-write of the code, I didn't make any
major design change in the look of the site itself. Part of that was in just wanting to limit the
scope of what I was changing in order to validate the new site worked and could easily be slipped
into place replacing the old code. I do want to go back and give the page a refreshed look but
haven't had time to do so yet.

I did get the code deployed using Serverless but it was done manually and anyone who knows me I am
all about that automation. I also really didn't like having to use Serverless which resulted in a
CloudFormation stack to deploy which doesn't fit with the rest of my deployment practices. At this
point this is simply because Terraform doesn't have some of the feature functionality that
Serverless is currently able to provide. I'm working on that though and will write about it when I
have it fleshed out a bit more. 

{{< highlight serverless >}}
    functions:
        website:
            handler: public/index.php
            description: 'GNU Privacy Guard Policy Manager'
            memorySize: 1024
            timeout: 28 # in seconds (API Gateway has a timeout of 29 seconds)
            layers:
                - ${bref:layer.php-80-fpm}
{{< /highlight >}}

The trick to fixing that is in writing a Terraform provider for Bref which will provide the same
functionality that Serverless has now. Bref provides a Serverless vendor plugin that allows it to
look up the current lambda layer version and fill in the proper ARN.

{{< highlight terraform >}}
    data "bref_lambda_layer" "php" {
        layer_name = "php-80-fpm"
    }

    resource "aws_lambda_function" "website" {
        filename         = data.archive_file.lambda_archive.output_path
        function_name    = "gpg-policy-website"
        role             = aws_iam_role.lambda_iam.arn
        handler          = "public/index.php"
        source_code_hash = data.archive_file.lambda_archive.output_base64sha256
        runtime          = "provided.al2"
        timeout          = 28
        layers           = [data.bref_lambda_layer.php.arn]
    }
{{< /highlight >}}

While this example isn't quite complete but gives the idea I'm working towards. I've started working
on the Bref provider and have an initial release I'm testing with but it is not complete yet. I'll
talk about it more when I've got it working like I want.

Another improvement I want to make is that the Lambda function currently works with my policy files
being build into another Lambda layer that is added along with the Bref PHP layer. My desire is to
take advantage of Lambda being able to mount an EFS volume that I can then implement my CI/CD
pipeline to publish the policy files to when I commit them to my repository. This will also work to
prove the concept for other projects I have on my planning board.

If you want to see the new site in action you can check out my [current policy][6]. As you can see
I still have a growing laundry list of things yet to do so I have a steady stream of project work
to spend my idle time on. I hope if you found this while searching for information you found this
helpful and will keep checking back for new tips and tricks.

[1]: https://vuejs.org/ "The Progressive JavaScript Framework"
[2]: https://angular.io/ "The modern web developer's platform"
[3]: https://symfony.com/ "Symfony, High Performance PHP Framework for Web Development"
[4]: https://laravel.com/ "The PHP Framework For Web Artisans"
[5]: https://bref.sh/ "Serverless PHP made simple"
[6]: https://undergrid.net/legal/gpg/policy/20140409/bd35f5ce24e2976afd9ce1e8bbac30346263b5ccae172e7b1f38f2d1c92ab918 "Current key policy"
