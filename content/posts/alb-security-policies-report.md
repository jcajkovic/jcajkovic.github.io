---
title: "LB security policy report"
date: 2023-03-17T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["aws", "lb", "python", "boto3"]
author: "Jozef Cajkovic, OpenAI ChatGPT"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Tutorial to create report of security policies configuration of LoadBalancers in AWS using https://github.com/pixel-systems/alb-security-policies-report"
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

If you're working with AWS Application Load Balancers (ALBs), you're probably familiar with the need to configure security policies to ensure that your traffic is secure. However, monitoring and managing these policies can be a challenge, especially as your environment grows more complex.

That's where [alb-security-policies-report](https://github.com/pixel-systems/alb-security-policies-report) comes in. This open-source repository, created by Pixel Systems, provides a set of tools and scripts that can help you monitor and report on the security policies of your ALBs. With alb-security-policies-report, you can:

- Generate detailed report on the security policies of your ALBs from multiple accounts.
- *(feature request) Identify any inconsistencies or non-compliance with your security requirements*

The repository includes of python script that use the AWS SDK (boto3) to extract data from your ALBs, which ist then used to generate report TXT format. 

One of the key benefits of alb-security-policies-report is its ability to contact multiple accounts when naming convention of spoke accounts IAM roles is followed. 

The tools and scripts provided can be customized to meet your specific needs, allowing you to generate reports that are tailored to your environment and requirements.

In conclusion, if you're working with AWS ALBs and want an easier way to monitor and report on your security policies, alb-security-policies-report is definitely worth checking out. With its robust set of tools and scripts, you'll be able to get a clearer picture of your ALB security and identify any potential issues before they become a problem.