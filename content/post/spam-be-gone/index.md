---
title: "Spam Be Gone"
type: post
author: jtbouse
date: 2008-07-09T03:09:11-05:00
tags:
  - dkim
  - domainkeys
  - phishing
  - sender id
  - sender policy framework
  - spam
  - spf
categories:
  - Projects
---
So I've got this serious jonesing love/hate relationship with spam. Personally I'd love to collect all the spammers of the world in a nice lead lined room and irradiate them with low yield nuclear waste. Not enough to kill them note you, just enough to ensure that they don't breed!

There are so many methods out there to try and curb the amount of spam out there but never seems to be enough adoption of them. I try my best to implement what I can as I've fallen victim of several "joe jobs" in the past. As a result of that I looked at Sender Policy Framework (SPF) before it got introduced into the IETF track and became spf2.0/mfrom,pra. I'm still running my spf1 classic records and it has helped a bit, but I wonder just how many servers really bother to check and honor the policies published.

Likewise, recently I've taken the time to implement DomainKeys and DKIM on our servers. On our newest domain which hasn't even been put to use yet so there are no email addresses in the wild I've actually been working to set the policy stating all emails will be signed. As I maintain the only legitimate servers that should be sending email out with the domain it shouldn't be a problem and should hopefully limit the use of it for phishing and spam.

Another tool in the aresenal that has helped considerably has be greylisting. It's simple, it works with existing protocols and I'm surprised more sites don't use it. So you incur a small delay in email delivery time but it eliminates untold amounts of spam just by simply delaying the inevitable. For the longest time greylisting and SPF were more than enough to help keep my inboxes clean of most spam although I still dealt with a handful or so.

The other tools I've found invaluable but some consider controversial is use of DNS RBLs. While not all RBLs are created equal there are some very good ones out there that have reputable organizations behind them. I do limit the number of RBLs I check but the ones I check eliminate a fair amount of the spam that once made it through to my mailbox. The composite RBLs are even nicer as they cover more with a single query which helps speed up performance when you have multiple RBLs along with DK, DKIM and SPF all making DNS queries as well.

A good metric that things were working for me was when I had it down to less than a handful of spam messages getting through to my inbox a day. I then updated one of my forwarded email addresses on a server that had no RBLs in use to my regular mailbox address. Within 24 hours the number of spam messages that had gotten through numbered in the hundreds. I worked quickly to get that address using the same blacklists as my main mail servers and the volume of spam nearly disappeared.

Bottom line is with existing technologies it's easy to curb the spam if more mail servers made use of them. When we can finally get authoritative authentication with the messages being delivered that will help even more. Although sadly as it's already shown all too well, as the technologies advance to reduce the spam the spammers themselves have evolved to get around them.

Never a slow day when you're wearing the postmaster/abuse hat but it comes with the job.
