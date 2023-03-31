---
title: "What's the difference between Route53 Alias and CNAME"
date: 2023-03-31T22:05:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["aws", "r53"]
author: "Jozef Cajkovic, OpenAI ChatGPT"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "R53 Alias vs. CNAME"
# canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/jcajkovic.github.io/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

Both Alias and CNAME records in Route 53 are used to map one domain name to another domain name. 

However, there are some important differences between the two:

**CNAME Record**: A CNAME (Canonical Name) record is used to create an alias that points one domain name to another. For example, if you create a CNAME record for "www.example.com" that points to "example.com", then anyone who types "www.example.com" in their browser will be redirected to "example.com". CNAME records can only be used to point to a domain name, not to an IP address.

**Alias Record**: An Alias record is similar to a CNAME record, but it has some important differences. An Alias record can be used to map one domain name to another, just like a CNAME record, but it can also be used to map a domain name to an AWS resource. This is because an Alias record is a Route 53-specific record type that works differently from a CNAME record. When you create an Alias record in Route 53, the domain name is automatically resolved to the IP address of the resource, without incurring the additional lookup time and cost of a CNAME record. Alias records can be used to map a domain name to an Elastic Load Balancer, an Amazon S3 bucket, an Amazon CloudFront distribution, or an AWS Elastic Beanstalk environment.

> In summary, while CNAME records are used to map one domain name to another, Alias records are used to map a domain name to an AWS resource. Alias records are specific to Route 53 and can provide some performance benefits over CNAME records.

Alias records have some advantages over CNAME records when pointing to an AWS Load Balancer.

One advantage of Alias records is that they provide better high availability for your applications. When you create an Alias record that points to an AWS Load Balancer, the record is automatically updated if the IP address of the load balancer changes. This means that your application will continue to be available even if the IP address of the load balancer changes due to scaling events, maintenance, or other reasons. This is because Route 53 uses the AWS resource record sets API to monitor the health of your load balancer, and automatically updates the Alias record to point to a healthy load balancer in the same region.

Another advantage of Alias records is that they provide better performance. When you use a CNAME record to point to an AWS Load Balancer, the DNS resolver first resolves the CNAME to the IP address of the load balancer, and then resolves the IP address to the actual instance. This can add additional latency to your application. With an Alias record, however, Route 53 resolves the record directly to the IP address of the load balancer, which can reduce the latency and improve the performance of your application.

Overall, if you are using an AWS Load Balancer to distribute traffic to your application, it's recommended to use an Alias record in Route 53 for better high availability and performance.