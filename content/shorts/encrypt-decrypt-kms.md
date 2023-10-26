---
title: "How to KMS Encrypt/Decrypt with aws-cli"
date: 2023-10-28T10:05:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["aws", "kms", "aws-cli"]
author: "Jozef Cajkovic, OpenAI ChatGPT"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Reader will learn how to use aws cli for kms encrypt and decrypt"
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

Decrypting a string with a KMS (Key Management Service) key using AWS CLI (Amazon Web Services Command Line Interface) is a process that involves several steps. Here's a step-by-step guide:

1. **Install and Configure AWS CLI**:
    - Before you can decrypt a string using AWS CLI, ensure that you have the AWS CLI installed and configured with the necessary access credentials.

2. **Encrypt the String**:
    - If your string is not yet encrypted, encrypt it using AWS KMS. You can use the following command:
      ```bash
      aws kms encrypt --key-id <key-id> --plaintext <string-to-encrypt> --query CiphertextBlob --output text | base64 --decode > <output-file>
      ```

3. **Decrypt the String**:
    - Use the following command to decrypt the encrypted string:
      ```bash
      aws kms decrypt --ciphertext-blob fileb://<path-to-encrypted-file> --query Plaintext --output text | base64 --decode
      ```

In the above steps:

- Replace `<key-id>` with the ID of your KMS key.
- Replace `<string-to-encrypt>` with the string you want to encrypt.
- Replace `<output-file>` with the name of the file where you want to save the encrypted string.
- Replace `<path-to-encrypted-file>` with the path to the file containing the encrypted string.

Here's a breakdown of the commands:

- In the encryption command, `--query CiphertextBlob --output text` extracts the encrypted string, which is then decoded from base64 and saved to a file.
- In the decryption command, `fileb://` is a prefix that tells AWS CLI to read the ciphertext blob from a file. `--query Plaintext --output text` extracts the decrypted string, which is then decoded from base64.

Make sure that the user you are authenticated as in AWS CLI has the necessary permissions to use the KMS key for decryption.

This is a simplified guide to get you started, but AWS KMS and AWS CLI have many other options and features that you might need to use depending on your specific use case. It's advisable to read the AWS CLI and AWS KMS documentation for more detailed information.