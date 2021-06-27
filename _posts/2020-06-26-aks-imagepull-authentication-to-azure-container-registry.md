---
layout: post
title: AKS imagePull authentication to Azure Container Registry
date: 2020-06-26 16:28:06.000000000 +01:00
type: post
---


A pod running in an AKS cluster and trying to pull an image from ACR needs Reader permissions on that ACR. 

The simplest way to do this is give the AKS cluster service principal Reader access on the ACR as follows: 


First, get the ID of the AKS cluster service principal (note, when you create an AKS cluster an SP is automatically generated):
{% highlight bash %}
AKS_SP_ID=$(az aks show \
  --resource-group <your aks resource group> \
  --name <your aks cluster name> \
  --query "servicePrincipalProfile.clientId" \
  --output tsv \
  --subscription <your aks subscription id>)
{% endhighlight %}

<br>
Now get the ID of your Azure Container Registry:
{% highlight bash %}

ACR_ID=$(az acr show --name <your acr name> \
  --resource-group <your acr resource group> \
  --query "id" \
  --output tsv \
  --subscription <your acr subscription id>)
{% endhighlight %}

<br>
Now grant your AKS cluster SP Reader rights on your ACR:
{% highlight bash %}
az role assignment create --assignee $AKS_SP_ID --role Reader \
  --scope $ACR_ID --subscription <your acr subscription id>
{% endhighlight %}

NOTE: Trying to pull a non existent tag can result in an authentication error (e.g. myreg.azurecr.io/docker-camunda:latest as opposed to myreg.azurecr.io/docker-camunda:myreg.azurecr.io/docker-camunda:0.0.10)


