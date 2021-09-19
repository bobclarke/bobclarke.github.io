---
layout: post
title: 
date: 2021-09-19 16:49
category: 
author: 
tags: []
summary: 
---

Resources
---
- https://www.vaultproject.io/api/secret/identity/group
- https://www.vaultproject.io/api/auth/userpass
- https://www.vaultproject.io/docs/secrets/identity#external-vs-internal-groups

Using the REST API
---
Retrieve your token 
```
vault token lookup
```

Use the token with curl 
```
curl --header "X-Vault-Token: s.xxxxxxxxxxxx" --request LIST https://127.0.0.1:8200/v1/auth/userpass/users
```

Manage local users
---
List
```
curl --header "X-Vault-Token: s.xxxxxxxxxxxx" --request LIST https://127.0.0.1:8200/v1/auth/userpass/users | jq .
```

Add / update
```
cat <<EOF | curl --header "X-Vault-Token: s.xxxxxxxxxxxxxxxx" --request POST https://127.0.0.1:8200/v1/auth/userpass/users/test.user --data @-
{
  "password": "superSecretPassword",
  "policies": "default"
}
EOF
```

Test the new account 
```
vault login -method=userpass username=test.user
```

Manage local groups
---
list
```
curl --header "X-Vault-Token: s.xxxxxxxxxxxxxxxx" --request LIST https://127.0.0.1:8200/v1/identity/group/name | jq .
```

read
```
curl --header "X-Vault-Token: s.xxxxxxxxxxxxxxxx" --request GET https://127.0.0.1:8200/v1/identity/group/name/"some group name" | jq .
```

create/update 
```
cat <<EOF | curl --header "X-Vault-Token: s.xxxxxxxxxxxxxxxx" --request POST https://127.0.0.1:8200/v1/identity/group --data @-
{
  "data":
  {
    "name":"test-group"
  }
}
EOF
```


How do groups map to policies
---
Check which policies have been mapped to a group
```
curl --header "X-Vault-Token: s.xxxxxxxxxxxxxxxx" --request GET https://127.0.0.1:8200/v1/identity/group/name/my-group -k | jq .
```

The output will be something like
```
{
  "request_id": "4c9c5a4d-7472-c70f-0397-58e8e1cad1da",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": {
    "alias": {},
    "creation_time": "2019-10-15T06:39:03.334062Z",
    "id": "bf86506b-6ce2-bde1-2bd1-8a4e54136372",
    "last_update_time": "2021-06-08T09:08:40.1640459Z",
    "member_entity_ids": null,
    "member_group_ids": null,
    "metadata": null,
    "modify_index": 7,
    "name": "my-group",
    "namespace_id": "root",
    "parent_group_ids": null,
    "policies": [
      "my-policy-001",
      "devops-team-policy",
      "sre-team-policy",
      "development-team-policy",
      "policy-002"
    ],
    "type": "external"
  },
  "wrap_info": null,
  "warnings": null,
  "auth": null
}
```

NOTE:  This is an okta group which is why the member_entity_ids field is empty

Okta groups vs local groups 
---
To read: https://learn.hashicorp.com/tutorials/vault/identity
This paragraph is useful 
By default, Vault creates an internal group. When you create an internal group, you specify the group members rather than group alias. Group aliases are mapping between Vault and external identity providers (e.g. LDAP, GitHub, etc.). Therefore you define group aliases only when you create external groups. For internal groups, you specify member_entity_ids and/or member_group_ids.

So, this is a little confusing....
- Under /v1/identity/group/name I can see groups that are external groups
- I assumed these were OKTA groups, however, when I added a policy to one of these groups the policy did not get applied to a user I know was a member of said group.
- Then under a different path (auth/okta/groups) I find what appears to be the real group 
```
curl --header "X-Vault-Token: s.xxxxxxxxxxxxxxxx" -k --request GET https://127.0.0.1:8200/v1/auth/okta/groups/devops-group | jq .
```

Note, you can also read this as follows
```
vault read auth/okta/groups/devops-group
```

From what I can the, the "real" group (i.e  /auth/okta/groups/devops) was not created anywhere in vault and has in fact been mapped automatically by Okta. The problem with that theory is that when I list the groups under /auth/okta/groups I only see a subset .. so I don't know how this subset is filtered out of the hundreds of group in okta
```
curl --header "X-Vault-Token: s.xxxxxxxxxxxxxxxx" -k --request LIST https://127.0.0.1:8200/v1/auth/okta/groups | jq .data

{
  "keys": [
    "devops",
    "sre",
    "prod-admins",
  ]
}
```

Re the other set of external group that were created by somebody, according to the docs the way these have to be mapped to an external backend like okta is via group alias within which the accessor for that backend is used as a "mount point" ... However I see no group aliases in our vault so I can only assume these groups are having no effect. (which was born out when I added a policy to one of these groups earlier) 


Policies
---
List 
```
curl --header "X-Vault-Token: s.xxxxxxxxxxxx" -k --request LIST https://127.0.0.1:8200/v1/sys/policies/acl | jq .
```

read
```
curl --header "X-Vault-Token: s.xxxxxxxxxxxx" -k --request GET https://127.0.0.1:8200/v1/sys/policies/acl/aor-nonprod-team | jq .data.policy
```


gnome-keyring-daemon and secret-tool
---
To display the vault token  
```
secret-tool lookup service vault.com
```

To update  (this will prompt for the value you with to store with a "Password:" prompt
```
secret-tool store --label foo  service vault.com
```

With this in mind we can generate a new token and assign any policy to it  (see https://learn.hashicorp.com/tutorials/vault/getting-started-policies)
```
vault token create -field token -policy=my-policy
```

Creating a new vault token and assigning a policy to it 
---

```
vault token create -field token -policy=devops-admin
```
