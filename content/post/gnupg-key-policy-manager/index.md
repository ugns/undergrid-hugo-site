---
author: jtbouse
categories:
- Projects
date: "2008-07-12T18:29:30Z"
guid: http://blog.undergrid.net/?p=5
id: 39
image: /wp-content/uploads/2018/08/board-928390_1920.jpg
tags:
- gpg
- key policy
- pgp
- Projects
- software
title: GnuPG Key Policy Manager
url: /2008/07/gnupg-key-policy-manager/
---
Taking GNU Privacy Guard key usage seriously I have had a published key usage policy that I embed the link into any GPG key signature when signing a key. After years of using PGP/GPG I have found that having an established usage and management policy is nice as it lets others know that you take your key usage seriously.

<!--more-->
Over the years the URL I&#8217;ve used for my key policy has changed, as well the URL itself has evolved with the times. Originally it was simply a text/plain file returned back. I have not turned it into a manager that not only presents the key policy, but also provides links to the detached signatures of the policy and verifies either the MD5 or SHA1 checksum of the policy file if provided.  This policy manager is currently being worked on to be able to provide as both a shareware and licensed versions. As it involves GPG key usage the license will be generated for the users GPG key.

Those interest can feel free to check out <a title="Jeremy T. Bouse GPG Key Policy" href="http://undergrid.net/legal/gpg/" target="_blank" rel="noopener">my key policy</a> and watch as it evolves. Those wishing to potentially make use of the policy manager can also inquire further as there is no current targeted date for a general release.