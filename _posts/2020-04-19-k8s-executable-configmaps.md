---
layout: post
title: K8s Executable ConfigMaps
date: 2020-04-19 15:47:17.000000000 +01:00
type: post
---

To inject a script into a Pod, it's simple enough to use a config map, however if you try to chmod the script to make it executable you'll get an error telling you that the file system is read-only.  This is easily solved by using ```defaultMode``` parameter

<img src="/assets/images/k8s-executable-configmaps1.png" width="600">

Here, I'm setting up a volume called pg-backup-script (which is pointing at a configMap named hadoop-ra-postgresql-backup-cm) and I'm setting it's filesystem permissions to -rwxr--r--. I'm then mounting it in the container under /tmp.

<img src="/assets/images/k8s-executable-configmaps2.png" width="600">


As can be seen, the configMap data has a "key" of backup.sh, which is now executable
