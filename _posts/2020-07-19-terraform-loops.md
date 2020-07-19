---
layout: post
title: Terraform loops with templates
date: 2020-07-19 15:32
category: 
author: 
tags: [terraform]
summary: 
---

One use case for looping in terraform is if you need to provide a list of values. The example below shows the installtion of the nginx-ingress Helm chart to a Kubernetes cluster. In this particular case we're setting up [TCP Services](https://kubernetes.github.io/ingress-nginx/user-guide/exposing-tcp-udp-services/) which requires a list of one of more arguments. We could just as easily provide a values file however the technique show below deomonstrates the use of a templating and looping nicely

{%highlight hcl%}
resource "helm_release" "ingress" {
  count       = 1
  name        = "ingress"
  repository  = "https://kubernetes-charts.storage.googleapis.com/"
  chart       = "nginx-ingress"
  namespace   = "foo3"

  values    = [
    local.tcpServicesConf
  ]

  set {
    name = "controller.ingressClass"
    value = "bobtest"
  }
}

locals{
  tcp_services = split(",",var.tcp_services_string)

  tcpServicesConf = <<-EOT
  tcp:
  %{ for line in local.tcp_services ~}
    ${line}
  %{ endfor ~}
  EOT
}

variable "tcp_services_string" {
  default = "8080: default/example-svc1:80"
}
{% endhighlight %}

<br>

When running:
```
terraform apply \
-var="tcp_services_string=4092: workflow/kcp-cp-kafka-0:9092,4093: workflow/kcp-cp-kafka-1:9092,4094: workflow/kcp-cp-kafka-2:9092" -auto-approve
```

The folling values are produced:
```
tcp:
    4092: "workflow/kcp-cp-kafka-0:9092"
    4093: "workflow/kcp-cp-kafka-1:9092"
    4094: "workflow/kcp-cp-kafka-2:9092"
```




