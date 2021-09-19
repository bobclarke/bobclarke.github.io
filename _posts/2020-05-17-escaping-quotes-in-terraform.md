---
layout: post
title: Escaping quotes in Terraform
date: 2020-05-17 13:33:20.000000000 +01:00
type: post
---
I had a slightly tricky issue recently whereby I needed to use a JQ expression as part of a local-exec.  The JQ expression needed quotes however Terraform wasn't happy with them.  It was a simple case of escaping with backslashes.

{% highlight bash %}
resource "null_resource" "confluent-kafka-env" {
  provisioner "local-exec" {
    when = "destroy"
    command = "ccloud environment list -o json | jq '.[] | select(.name | contains(\"${var.env-name}\")) .id'"
  }
  provisioner "local-exec" {
    command = "ccloud environment create ${var.env-name} -o json >> env_id.json"
  }
}
{% endhighlight %}
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTM4NDI1NjUwOV19
-->
