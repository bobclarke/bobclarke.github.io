---
layout: post
title: 
date: 2020-09-14 13:47
category: 
author: 
tags: []
summary: 
---


Let's say you've set up two listeners, one internal and one external as follows
```
KAFKA_LISTENERS=INTERNAL://0.0.0.0:9092,EXTERNAL://0.0.0.0:19092
```

To test these, you can use kafkacat<br>
First, let's test the external listener from our local machine<br>
Step one is to get the metadata
```
kafkacat -b <extrnal IP>:<external port> -L
```
For example:
```
kafkacat -b kafka.stack1.net:2000 -L
```
As you can see in this exmaple I've mapped the external listener to an IP and port. In my case I'm running kafka inside a Kubernetes cluster and I explain how to set up the networking in [THIS ARTICLE - TBC]

This will potentially send back alot of info if you already have topics defined so you may want to filter as follows:
```
kafkacat -b kafka.stack1.net:2000 -L | grep -v partition 
```
The response should look something like this
```
Metadata for all topics (from broker 0: kafka.stack1.net:2000/0):
 3 brokers:
  broker 0 at kafka.stack1.net:2000
  broker 2 at kafka.stack1.net:2002
  broker 1 at kafka.stack1.net:2001 (controller)
  ```

So, the above is testing just half of the picture. I.e it's testing whether we can hit a broker and retrieve metadata about the all the brokers in the cluster. 

Next we need to check if we can publish and consume stuff

To do this, open two terminal windows, one will be for publishing, the other will be for consuming.

In window 1 type: ```kafkacat -b kafka.stack1.net:2000 -P -t MY_TEST_TOPIC``` then press return

In window 1 type: ```kafkacat -b kafka.stack1.net:2000 -C -t MY_TEST_TOPIC``` then press return

Now go back to window 1 and type a message, Hello World is always a good one :)

You should see the message appear in window 2


