---
layout: post
title: 
date: 2020-07-20 17:20
category: 
tags: []
summary: 
---
Consider the following stateful set:
{%highlight bash%}
> kubectl get pods | grep kafka
workflow01-dev-cp-cp-kafka-0                           2/2     Running            3          6h38m
workflow01-dev-cp-cp-kafka-1                           2/2     Running            2          6h37m
workflow01-dev-cp-cp-kafka-2                           2/2     Running            0          6h36m
{%endhighlight%}

Which has the following associated headless service:
{%highlight bash%}
> kubectl get svc | grep headless
workflow01-dev-cp-cp-kafka-headless       ClusterIP   None           <none>        9092/TCP            6h49m
{%endhighlight%}

From within a pod on the cluster, the DNS looks like this:
{%highlight bash%}
> dig workflow01-dev-cp-cp-kafka-headless.workflow01-dev.svc.cluster.local +short
10.1.1.49
10.1.2.122
10.1.0.124
{%endhighlight%}

This is returning the IP's of all three stateful set pods (this can be verified as below)
{%highlight bash%}
> kubectl get pods -o wide  | grep kafka
workflow01-dev-cp-cp-kafka-0      2/2     Running    3  6h53m   10.1.2.122   aks-default-27860648-vmss000002
workflow01-dev-cp-cp-kafka-1      2/2     Running    2  6h51m   10.1.1.49    aks-default-27860648-vmss000001
workflow01-dev-cp-cp-kafka-2      2/2     Running    0  6h50m   10.1.0.124   aks-default-27860648-vmss000000
{%endhighlight%}

If I want to get the address of a single member of the stateful set:
{%highlight bash%}
> dig workflow01-dev-cp-cp-kafka-0.workflow01-dev-cp-cp-kafka-headless.workflow01-dev.svc.cluster.local
10.1.2.122
{%endhighlight%}






