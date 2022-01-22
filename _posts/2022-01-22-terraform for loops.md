---
layout: post
title: 
date: 2022-01-22 17:19
category: 
author: 
tags: []
summary: 
---

I had a challenge recently whereby I had to automate creating an Azure Private Link Service and associate it with an existing AKS (Azure Kubernetes Service) load balancer.  What made this difficult is that the load balancer had multiple Front End IP configurations associated with it (this is because multiple k8s services of type LoadBalancer will often be associated with a single load balancer in the form of multiple Front End IP Configurations)

To solve this problem I needed to use the following logic 
- Get the LoadBalancer IP address of the k8s service you're interested in
- Get the list of front end IP configs from the load balancer
- Create the PrivateLink  service and associate it with front end ip config (and it's subnet) with the matching IP address

The trick here was get the load balancer config using a datasource, build a map from it using the FE config IP's as key's, and then use that key to match the IP address on the Load Balancer service (which in my case were sitting in front of ingress controllers however this would with at k8s service of type LoadBalancer. 

Here's the code ...

```
//===================================================================
// vars 
//===================================================================
variable "subscription_id" {
  default = "xyz"
}

variable "client_id" {
  default = "xyz"
}

variable "client_secret" {
  default = "xyz"
}

variable "tenant_id" {
  default = "xyz"
}

variable "admin_password" {
  default = "xyz"
}

variable "context" {
  default = "xyz"
}

variable "location" {
  default = "xyz"
}

variable "resource_group_name" {
  default = "xyz"
}

//===================================================================
// Provider setup 
//===================================================================
provider "azurerm" {
  version = ">= 2.41.0"
  subscription_id = var.subscription_id
  client_id       = var.client_id
  client_secret   = var.client_secret
  tenant_id       = var.tenant_id
  features {
  }
}

provider "kubernetes" {
  config_path    = "~/.kube/config"
  config_context = var.context
}


//===================================================================
// Get the ingress controller details
//===================================================================
data "kubernetes_service" "internal-ingress" {
  metadata {
    name = "nginx-ingress-controller"
    namespace = "ingress"
  }
}

locals {
  ingress_ip = data.kubernetes_service.internal-ingress.status.0.load_balancer.0.ingress.0.ip
}

//===================================================================
// Build a map of Load Balancer frontend IP configs keyed on IP address
//===================================================================
data "azurerm_lb" "lb" {
  name                = "kubernetes-internal"
  resource_group_name = var.resource_group_name
}

locals {
  feconfigs = {
    for feconfig in data.azurerm_lb.lb.frontend_ip_configuration:
      "${feconfig.private_ip_address}" => {
        "subnet_id" = feconfig.subnet_id
        "load_balancer_frontend_ip_configuration_id" = feconfig.id
      }
    }
}

//===================================================================
// Now we can retrieve the FE config and Subnet ID's for this IP 
//===================================================================

locals {
  lb_feconfig = local.feconfigs[local.ingress_ip].load_balancer_frontend_ip_configuration_id
  lb_subnet = local.feconfigs[local.ingress_ip].subnet_id
}

output "subnet" {
  value               = local.lb_subnet
}

output "feconfig" {
  value               = local.lb_feconfig
}

//===================================================================
// Add the PrivateLink Service
//===================================================================
resource "azurerm_private_link_service" "pl-service" {
  name                = "pl-service"
  location            = var.location
  resource_group_name = var.resource_group_name

  auto_approval_subscription_ids              = [var.subscription_id]
  visibility_subscription_ids                 = [var.subscription_id]

  nat_ip_configuration {
    name      = "nat_ip_config"
    primary   = true
    subnet_id = local.lb_subnet
  }

  load_balancer_frontend_ip_configuration_ids = [
    local.lb_feconfig,
  ]
}
```

The super cool thing I learned here was that you can do really useful loops within a locals block as follows... 
```
data "azurerm_lb" "lb" {
  name                = "kubernetes-internal"
  resource_group_name = var.resource_group_name
}

locals {
  feconfigs = {
    for feconfig in data.azurerm_lb.lb.frontend_ip_configuration:
      "${feconfig.private_ip_address}" => {
        "subnet_id" = feconfig.subnet_id
        "load_balancer_frontend_ip_configuration_id" = feconfig.id
      }
    }
}
```

The above creates a map as follows...
```
feconfigs = {
  "10.144.6.127" = {
    "load_balancer_frontend_ip_configuration_id" = "/subs/123/rg/foo/providers/ms/lbs/k8s/feIPConfig/fe02"
    "subnet_id" = "/subs/123/rg/foo/providers/ms/virtualNetworks/bar/subnets/sn02"
  }
  "10.224.72.4" = {
    "load_balancer_frontend_ip_configuration_id" = "/subs/123/rg/foo/providers/ms/lbs/k8s/feIPConfig/fe05"
    "subnet_id" = "/subs/123/rg/foo/providers/ms/virtualNetworks/bar/subnets/sn05"
  }
  "10.144.6.5" = {
    "load_balancer_frontend_ip_configuration_id" = "/subs/123/rg/foo/providers/ms/lbs/k8s/feIPConfig/fe01"
    "subnet_id" = "/subs/123/rg/foo/providers/ms/virtualNetworks/bar/subnets/sn101"
  }
  "10.224.72.9" = {
    "load_balancer_frontend_ip_configuration_id" = "/subs/123/rg/foo/providers/ms/lbs/k8s/feIPConfig/fe04"
    "subnet_id" = "/subs/123/rg/foo/providers/ms/virtualNetworks/bar/subnets/sn04"
  }
  "10.223.61.12" = {
    "load_balancer_frontend_ip_configuration_id" = "/subs/123/rg/foo/providers/ms/lbs/k8s/feIPConfig/fe03"
    "subnet_id" = "/subs/123/rg/foo/providers/ms/virtualNetworks/bar/subnets/sn103"
  }
}
```

