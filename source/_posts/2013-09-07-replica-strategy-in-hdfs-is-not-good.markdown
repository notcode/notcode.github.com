---
layout: post
title: "Replica strategy in hdfs is not good enough"
date: 2013-09-07 19:02
comments: true
categories: 
- hadoop
---


##### Two methods of Replica

I've known two methods of replica in distributed storage system.
One is like hadoop, which distributes the three replicas of a block on three different nodes.
The other is like mongodb, which store data into shardings with three nodes in each sharding.These three nodes in the same sharding are identical.

Both of the two strategy has a "at most two dead node" tolerance,that meanings if one or two node in cluster crash,all the data is still available. But what happen to three nodes?

<!-- more -->


##### Think it for a while

Suppose there are N nodes and B blocks in cluster. What's the probability of availability with three nodes lost?

* probability in hadoop cluster
  
For one block,the probability that not all three replicas of this block is on the three dead nodes is:
    1-6/[n(n-1)(n-2)]
  
The cluster is available with three node lost means all block is available,and the probability is :
    (1-6/[n(n-1)(n-2)])^b


* probability in mongodb cluster

The nodes number is N, shardings number should be N/3. The probability that cluster is not available is 1/[(N/3)(N/3)],
and the opposite is :
    1-1/[(N/3)(N/3)]



##### Which is better

  If N=20,B=10k,which is better?
  Actually when there are 20 nodes in a cluster,the number of blocks is far greater than 10K.

* probability in hadoop is: 0.0154%
* probability in mongo is: 99.7%

See,the tolerance in hadoop is not good enough.
