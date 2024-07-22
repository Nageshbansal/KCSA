ref: https://kubernetes.io/docs/reference/access-authn-authz/authentication/

### Users In Kubernetes

Two type of users:
- Normal Users
- Kubernetes Service Accounts

Cluster independent service manages users in following ways:
- private keys ( distributed by administrator )
- User Store ( google accounts, keystone)
- file list with usernames and passwords

```
Kubernetes does not have objects which represents/stores normal user accounts i.e Normal users can't be added to a cluster through API Call
```

- if  a user presents a valid certificate signed by the cluster's CA is considered authenticated, and the Kubernetes determine the username from common name field in `subject` of the cert. From there , RBACs sub-system would determine whether the user is authorized to perform specific operations on a resource.

- In contrast, service accounts are users managed by the Kubernetes API. They are bound to specific namespaces, and created automatically by the API server or manually through API calls. Service accounts are tied to a set of credentials stored as `Secrets`, which are mounted into pods allowing in-cluster processes to talk to the Kubernetes API.

So Api requests are tied to be from:
- Normal User
- Service account
- [Anonymous Requests](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#anonymous-requests)


### Authentication Strategies

Kubernetes uses client certificates, bearer tokens, or an authenticating proxy to authenticate API requests through autheticating plugins.
 As HTTP requests are made to the API server, plugins attempt to associate the following attributes with the request:


| field        | description                                                                                          | example                     |
| ------------ | ---------------------------------------------------------------------------------------------------- | --------------------------- |
| username     | a string which identifies the end user                                                               | kube-admin , xyz@efg.com    |
| UID          | a string which identifies the end user and attempts to be more consistent and unique than username   |                             |
| groups       | set of strings, each of which indicates the user's membership in a named logical collection of users | system:masters, devops-team |
| extra fields |  a map of strings to list of strings which holds additional information authorizers may find useful. |                             |

```
Note: you can use as many authentication methods at once. you should usually use atleast two methods
- service account tokens for service accounts
- at least one other method for user authentication.
```

When multiple authenticator modules are enabled, the first module to successfully authenticate the request short-circuits evaluation. The API server does not guarantee the order authenticators run in.


