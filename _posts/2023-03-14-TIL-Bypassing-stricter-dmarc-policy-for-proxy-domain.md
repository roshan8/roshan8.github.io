---
title:  "TIL: Bypassing stricter dmarc policies while sending emails from a proxy domain"
layout: post
---

## Usecase
We were hosting status pages on behalf of our customers. And there is also an option for users of our customers can subscribe for these status page updates. So the customer was pointing their status page subdomain to our IPs.  

Things did work well for most of our customers. However, a few have a stricter `dmarc` policy on their domain to reject emails if the sender doesn’t have either SPF/DKIM record set. For eg: imagine a customer has a status page with us. And their status page domain is `status.example.com` and their dmarc set to following
`v=DMARC1; p=reject; pct=100; rua=mailto:it@example.com;"`  

Since customers haven’t set either SPF/DKIM pointing to opsmonk.dev, Emails might not be delivered to uses who subscribed for “status.example.com” updates. So there are 2 workarounds. 

### Solution 1: Set spf record for status.example.com
Customers can set SPF records pointing to us (i.e opsmonk.dev)
They can add a TXT record on their subdomain “status” set to the following
```sh
v=spf1 include:incidents.opsmonk.dev ~all
```
We used Mailgun provider to send emails to all our customers. Incidents.opsmonk.dev spf records were pointing to Mailgun. So customers including our domain in their spf record indirectly includes IPs set by mailgun 

### Solution 2: Loosen up dmarc policy not on base but just on sub domain
Customer can change their dmarc policy on a subdomain(“status.example.com”) level. So it can take precedence over the policy set on the base domain. 
```sh
v=DMARC1; p=none; pct=100; rua=mailto:it@example.com;"
```

#### The latter solution is not recommended as an attacker can misuse this and start sending emails from your domain. 