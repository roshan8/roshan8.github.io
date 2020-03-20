---
title:  "Leveraging Cloudflare to improve the performance and security of your site"
layout: post
---

![Cloudflare](https://www.bleepstatic.com/content/hl-images/2019/09/29/cloudflare-new-logo.jpg)

## What is Cloudflare?
Cloudflare is a CDN which acts as a middle layer between your actual hosting provider and the user browsing your website. Cloudflare has datacenter over 200 location where they can serve webpages from these edge locations.

## Why should we use Cloudflare

# Free and easy to setup:

It's very easy to get started with Cloudflare. and a lot of services offered as part of `free plan`

## DDoS Protection

The DDoS attack became very common in recent years when many vendors offering DDoS as a [service](https://securelist.com/the-cost-of-launching-a-ddos-attack/77784/).  Cloudflare was the pioneer in offering DDOS protection for free. It mitigates DDoS attacks, including those that target UDP and ICMP protocols, SYN/ACK, DNS, and NTP amplification and Layer 7 attacks. The best part is, Cloudflare can ensure genuine traffic will still hit your servers during DDoS attacks

## World’s fastest DNS resolver (1.1.1.1)

Cloudflare offers a free and fastest public DNS resolver in the world. ![Query-speed](https://www.cloudflare.com/img/learning/dns/what-is-1.1.1.1/query-speed.png)
ISP’s can always snoop on your traffic and send ads based on your internet usage. Cloudflare DNS server has Dnssec protection. It even offers a mobile app in case if you want to protect your traffic on your [mobile phones](https://1.1.1.1/dns/)

## Security

- When proxy enabled, Cloudflare can mask your actual server IP addresses. So it’s difficult for hackers to attack the servers.
- Cloudflare offers free SSL, multiple TLS encryption mode is available
![TLS-Encryption](https://docs.bitnami.com/images/img/platforms/common/cloudflare-SSL.png)
- Cloudflare supports HTTPS redirection, HTTP Strict Transport Security (HSTS)
 and minimum TLS version enforcement so SRE’s no need to worry about this on a server level.
- Support for WAF(web application firewall). Cloudflare regularly updates it's rulesets for the newest threats, So customers no need to update rulesets regularly. However, Cloudflare WAF is only available from the PRO plan.

## Smart routing with Argo

[Cloudflare Argo](https://www.cloudflare.com/products/argo-smart-routing/) smart routing can detect real-time network congestion and route web traffic across the fastest and most reliable network paths. Cloudflare claims web assets perform 30% faster

## Analytics:

Cloudflare provides deeper insights into incoming traffic. Default graphs include total requests, web traffic requests by country, bandwidth, unique visitors, threats, rate-limiting, traffic served over SSL, firewall etc.
![Cloudflare-analytics](https://www.cloudflare.com/resources/images/slt3lc6tev37/4rXajXOhL1JZaor9cIUbtX/47c4f0e8a5d800325942f01f728cdf41/Dashboard_Firewall.png)

## Cloudflare load balancers:

[Cloudflare load balancers](https://www.cloudflare.com/en-in/load-balancing/) are pretty useful when using a multi-cloud strategy. 

## Cheapest Domain Registrar

Cloudflare recently announced that they will be [selling domain](https://blog.cloudflare.com/cloudflare-registrar/) at a wholesale price each TLD charges. Although currently domain transfer from other registrars to Cloudflare is supported. Directly buying a domain from Cloudflare is not yet supported. However, not all the TLD’s are supported yet. Please check [this](https://www.cloudflare.com/tld-policies/) supported TLD’s




