---
layout: post
title: Paperclip S3 AccessDenied
date: 2015/10/06
---

# Aws::S3::Errors::AccessDenied (Access Denied):

Assuming you are using AWS's IAM and you created a dedicated User for these uploads.

If you get this error when trying to upload to S3, you need to assign this IAM User the "AmazonS3FullAccess" Policy.

<img src="/images/access-denied-iam-user-amazons3fullaccess-policy.png" alt="accessdenied iam user amazons3fullaccess policy" width="700"></img>


