---
title: AWS S3 as FTP and policy granularity issues
language: default
date: 2015-07-09 15:02:46
tags:
  - AWS
  - dev
---

At the company where I work, we've procured a consultancy to do some work on the upgrade and merging of 2 separate [Kentico](http://www.kentico.com/) websites. Makes total sense to outsource this work as they've got the expertise and "CMS" isn't a core capability of my workplace. Anyway they need the content and databases for both sites, which sums to about 10 gig in total. They don't have an FTP server and the suggestion was made to physically deliver this data securely via traditional means...

I didn't think much of this suggestion, so I took matters into my own hands and created an [AWS S3](http://aws.amazon.com/s3/) bucket with the intent that the consultancy could access this bucket (and only this bucket) using something like [CloudBerry Explorer](http://www.cloudberrylab.com/free-amazon-s3-explorer-cloudfront-IAM.aspx), or [S3Fox](http://www.s3fox.net/). I created a user for the consultancy in [IAM](http://aws.amazon.com/iam/), saved the access/secret key pair, and went to work on defining and attaching a policy to this user. I tried a couple different policies, but I couldn't see any S3 buckets! When I used my own access key, everything was good and I could see everything I was supposed to. Eventually, I just used the standard [AmazonS3FullAccess](http://docs.aws.amazon.com/directoryservice/latest/adminguide/role_s3_full_access.html) policy and what do you know, I could see everything! This, however, wasn't a viable solution, as the consultancy could see and get to the contents of ALL our other S3 buckets. I did some deeper investigation and came upon an [article on StackOverflow](http://stackoverflow.com/questions/6615168/s3-policy-limiting-access-to-only-one-bucket-listing-included), which perfectly outlined my problem.

Turns out there no way to let the user list only selected buckets. You can either list all buckets or none. This seems like a bit of an oversight by Amazon in my opinion.