---
layout: post
title: "Be careful with mapreduce while writing into database"
date: 2013-10-13 10:04
comments: true
categories: 
- hadoop
---

### A Strange Case

Recently we come across a problem when writing into database in mapreduce jobs. 
The job gets lots of failure attempt, though succeeds finally.
We reviewed the source code of job, nothing seemed suspicious, there must be something wrong in our cluster.

So we reproduce the scenario.

<!-- more -->

### Crime scene investigate

##### Let's check the log at first.

From the web console of jobtracker, we find some error logs emitted by the failure attempts.
The logs tell that inserting operation can't be commited because there is already a row in the table, with same primary key.

##### What about the job side?

The job writing into database is a map-only job, and the written table has a primary key on multiple columns. 
The input data is guaranted unique on the primary key, so there should be no conflits while executing sql like `'insert into ...'`

We notice that all of the failure attempts's id is greater than 0, that remind me a feature called peculative task in hadoop.
The insert confliction is caused by this feature. 

Bingo! We find the killer!

### Close the case

After close speculative task by `job.setMapSpeculativeExecution(false)`, the job succeeds silently.

How lucky that the table is created with a primary key!
Otherwise, we will get multiple occurence of the same record silently.
