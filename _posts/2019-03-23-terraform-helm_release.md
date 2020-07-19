---
layout: post
title: Providing values for a Terraform helm_release
date: 2019-03-23 22:42:03.000000000 +00:00
type: post
---
There are multiple ways to provide values for a helm chart when using the helm terraform provider. Three examples are show below (highlighted in red): the first pointing at a file, the second using a Here Doc, the third using a Set.

**NOTE**: the `<<-` syntax removes whitespace whereas using the `<<` syntax (without the hyphen "-") does not

{%highlight hcl%}
resource "helm_release" "brigade" {
    count          = "${var.enabled}"
    name           = "brigade"
    namespace      = "${var.namespace}"
    chart          = "brigade"
    
    values         = ["${file("${path.module}/config/values.yaml")}"]
}
{%endhighlight%}


{%highlight hcl%}
resource "helm_release" "brigade" {
    count          = "${var.enabled}"
    name           = "brigade"
    namespace      = "${var.namespace}"
    chart          = "brigade"
    
    values         = [
        <<-EOF
        brigade-github-app:
          enabled: true
        EOF
    ]
}
{%endhighlight%}


{%highlight hcl%}
resource "helm_release" "brigade" {
    count          = "${var.enabled}"
    name           = "brigade"
    namespace      = "${var.namespace}"
    chart          = "brigade"
    
    set{
        name       = "brigade-github-app.enabled"
        value      = "true"
    }

    set{
        name       = "ingress.enabled"
        value      = "true"
    }
}
{%endhighlight%}
