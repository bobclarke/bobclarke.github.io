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
https://docs.microsoft.com/en-us/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-purge

#### deleting a key-vault 
---
list
```
az keyvault list --resource-type vault  --subscription my-test-keyvault
```

delete 
```
az keyvault delete  --name my-test-keyvault  --subscription my-test-subscription
```

purge 
```
az keyvault purge  --name my-test-keyvault  --subscription my-test-subscription
```

make sure it's not in a soft deleted / recoverable stage 
```
az keyvault list-deleted --resource-type vault --subscription  my-test-subscription
```


