---
layout: post
title: Run, and exec into, a kubernetes pod on the fly
date: 2019-11-09 21:25:37.000000000 +00:00
type: post
---

{%highlight bash%}
kubectl run -it test-pod --rm --image busybox --restart Never -n <namespace> -- /bin/sh
{%endhighlight%}
