---
layout: post
tags:
- aws
- S3
- Migration
- Database
categories:
- AWS
date: 2019-07-04 01:00:00 +1100
title: Migrating 6.5TB of FileStream Data to S3 - A journey concluded

---
So, this week concludes our migration to S3 for this component of our system, and it's been a rather epic journey. As a sigh of relief, I made a [post on reddit](https://www.reddit.com/r/aws/comments/c7rygt/finally_finished_a_65tb_database_s3_i_can_sleep/), and was overwhelmed with the response. Over 100 upvotes and 65+ comments, was rather unexpected. 

From the comments, I thought maybe this was a good excuse to return to writing on this blog, and document the migration and issues we dealt with and how we overcame them for all to (maybe) read. 

### The problem

Where did we start? The system we work with is approximately 15 years from inception now. We've grown a LOT in that time. I've been with the business for 13 of those years, so seen and been involved in a lot of it. Sadly, with all things old, comes technical debt. 

About 2 years ago, we made the decision to migrate to AWS. This in itself was a massive undertaking, due to a lot of segregated components, old legacy and non-legacy systems with under-documented interactions, and at the same time, 