---
title: "Wahoo to Garmin sync lambda function"
date: 2023-04-08T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["aws", "lambda", "python", "terraform", "wahoo", "garmin"]
author: "Jozef Cajkovic, OpenAI ChatGPT"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "How I sync my bike activities from Wahoo ecosystem to Garmin ecosystem via serverless lambda solution."
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

prerequisities:
- wahoo device, wahoo account
- dropbox account
- garmin connect account

- Create Lambda function
- Create DropBox App