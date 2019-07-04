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

#### The problem

Where did we start? The system we work with is approximately 15 years from inception now. We've grown a LOT in that time. I've been with the business for 13 of those years, so seen and been involved in a lot of it. Sadly, with all things old, comes technical debt. 

About 2 years ago, we made the decision to migrate to AWS. This in itself was a massive undertaking, due to a lot of segregated components, old legacy and non-legacy systems with under-documented interactions, and at the same time, rather rapid growth, both in terms of staff, but also clients and the system(s) themselves. 

We set a goal to have our primary systems migrated within 12 months. We missed that by another 6 months, but at the 18 month timeline, we managed to cut-over. That's probably a story for another day, but it was a rather long journey, as we had intended to not just "Lift and Shift", but to re-engineer where we could. We wanted to at least leverage auto-scaling groups, multi-AZ presence, lamba's in some areas, some of AWS Security measures like the ALB and WAF, CloudFront where we can, and more. For the most part, we managed to achieve this, which considering we still have a lot of the legacy components in operating, this was no small feat. 

The big fish here though: All our system runs off a central Microsoft SQL database. Nothing special there, and that moved to RDS without a problem. Where we (expected, and) had problems, is with what we called our Paperwork Database. This was a second Microsoft SQL Server, hosted on premises, that was approximately 6.5TB of binary objects stored in FileStream format within the database. For those that don't know it, this allows the SQL Server to store the binary blob as a GUID in the file system, rather than as LOB within the database MDF itself. Can lead to less chance of corruption from single multi-TB sized files, and also offers some other advantages with direct file access that we didn't use. 

Now, not only could this database not directly migrate to RDS (FileStream is not supported on RDS), cost-wise, it was going to be one helluva stupid move. We evaluated the idea of using an EC2 instance, but decided against it for 2 reasons:

1. Cost. EC2 doesn't cost as much as RDS for storage, but it's still significantly higher than S3 costs.
2. "as-a-service" - Part of the drive into AWS was the managed-services and their offerings. Essentially, pay more for the service, but reduce the need to manage it. Since we were heavy on developers, and light on the Sys-Admin side of things, this made a lot of sense for us, and still does. 

#### Timeline

So, we had a 12 month goal for everything _except_ the paperwork database. We had highlighted early on that this was going to be a much bigger effort. We overshot by 6 months, but overall I was still pretty happy with that. I set a goal of 3 further months post-migration to complete the paperwork migration, and sadly, it's been just on 7 months, so overshot, but not by too much. 

**Driving Pressures**:

Costs, again, are a factor here. We plan to shut-down our on premises (Co-Location) servers, so this was a big blocker to doing so, and every month this took to migrate, was another month paying for the co-location. 

Backups were the other. While we had a rather decent infrastructure for backups (Veeam, 2 co-located servers for primary and archival backups, plus off-site), our primary infrastructure was questionable (don't get me started on hyper converged systems!), and this particular backup had been starting to give me worries. Veeam/VSSS was starting to choke on the backups (Remember how I mentioned FileStream puts every binary blob into a file on the file system, when you have many millions of rows, this means many millions of files -- VSSS doesn't seem to like this at a certain point). We'd already progressed past the point of blowing out timeouts for VSSS Freezes, and disabling guest indexing. Backup jobs took > 14 hours to do now, and failures of the job was becoming more and more frequent. 

We also calculated the speed of restoring from our backup servers as being quite low (network constraints, relatively low performance disks), and could have been facing 1+ weeks to restore the full backup! This would have been disastrous to the business. 