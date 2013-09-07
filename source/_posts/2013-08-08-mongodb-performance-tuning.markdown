---
layout: post
title: "mongodb performance tuning"
date: 2013-08-08 09:47
comments: true
categories: 
- mongodb
---

#### 1.背景

mongodb的写性能很一般,即使采用bulk insert，也达不到我们的业务需求。
另外，mongodb有一个DB粒度的读写锁,因此采用并发写也无法提高写性能。因为读写锁的关系，在有写负载较高的情况下,读性能也被拉低了。

引自mongodb官网:
> Beginning with version 2.2, MongoDB implements locks on a per-database basis for most read and write operations. 


<!-- more -->

#### 2.如何优化

折腾了两天，最终的优化方案如下：

* 使用replica set，读写分离

 replica set可以避免读写竞争，保证mongodb正常的读写性能。而且为了HA，replica set是必须要做的，所以没有测试replica set对性能影响。

* 对collection做shard，提高写性能

做shard能近似线性的提高写速度，在做线性扩展时，可以通过增加shard来达到预期。
但要如果想将写速度提高1~2个数量级，shard数量要x10或x100，代价太高。

* 合并document,减少insert次数

终极大招还是这条策略。将原来的document10个一组,合并成一个document，insert的次数降了一个数量级，写速度便不再是问题。

#### 3.shard benchmark

对shard做了比较简单的测试，只是验证一下shard的效果。结果如下：

写进程并发数:8

* 没有shard,7000 inserts/sec

* 2个shard,12000 inserts/sec

* 4个shard,16000 inserts/sec

测试过程中，mongodb的写速度不是很稳定。

*测试代码:*

    var mongo=require('mongodb');
    var util=require('util');
    var fs=require('fs');
    var STEP=require('./step');

    var collection;
    function loop(){
        var d=new Date();
        var s=100000;
        STEP(   
            function(){
                var group=this.group();
                for(var i=0;i<s;i++){
                    collection.insert({k:i},group());
                }
            },
            function(err,res){
                if(err){throw err;}
                console.log('TPS:\t'+s*1000.0/(new Date()-d))
                setTimeout(loop,10);
            }
        );  
    }   
    mongo.connect('mongodb://mongo.test.clouds.com:27017/x?w=1',function(err,db){
        if(err){throw err;}
        db.collection('a',function(err,c){
            if(err){throw err;}
            collection=c;
            loop();
        });
    });


