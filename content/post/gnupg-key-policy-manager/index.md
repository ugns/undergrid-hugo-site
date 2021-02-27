---
title: GnuPG Key Policy Manager
featured_image: images/professional-solutions.jpg
type: post
author: jtbouse
date: "2008-07-12T18:29:30Z"
tags:
- gpg
- key policy
- pgp
- Projects
- software
categories:
- Projects
series:
- Gnu Privacy Guard
---
Taking GNU Privacy Guard key usage seriously I have had a published key usage policy that I embed
the link into any GPG key signature when signing a key. After years of using PGP/GPG I have found
that having an established usage and management policy is nice as it lets others know that you take
your key usage seriously.

Over the years the URL I've used for my key policy has changed, as well the URL itself has evolved
with the times. Originally it was simply a text/plain file returned back. I have not turned it into
a manager that not only presents the key policy, but also provides links to the detached signatures
of the policy and verifies either the MD5 or SHA1 checksum of the policy file if provided.  This
policy manager is currently being worked on to be able to provide as both a shareware and licensed
versions. As it involves GPG key usage the license will be generated for the users GPG key.

Those interest can feel free to check out [my key policy](https://undergrid.net/legal/gpg/) and
watch as it evolves. Those wishing to potentially make use of the policy manager can also inquire
further as there is no current targeted date for a general release.
