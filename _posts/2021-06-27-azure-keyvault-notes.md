---
layout: post
title: 
date: 2021-06-27 14:42
category: 
author: 
tags: []
summary: 
---

#### Reference doc
---
<https://docs.microsoft.com/en-us/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-purge>

#### deleting a key-vault 
---


list all keyvaults of type vault (as opposed to hsm) in the subscription "my-test-subscription"
```
az keyvault list --resource-type vault  --subscription my-test-subscription
```


delete the keyvault (this will delete it but it'll still be recoverable)
```
az keyvault delete  --name my-test-keyvault  --subscription my-test-subscription
```


purge (this will actually delete it and it will no longer be recoverable)
```
az keyvault purge  --name my-test-keyvault  --subscription my-test-subscription
```


make sure it's not in a soft deleted / recoverable stage 
```
az keyvault list-deleted --resource-type vault --subscription  my-test-subscription
```


