#SecurityFundamentals

ref: https://kubernetes.io/docs/concepts/configuration/secret/

Secret is an object that contains a small amount of sensitive data such as password, token or a key - so that you dont have to put the confidential data in application code/pod specification/container image

Secrets are similiar to configMaps but are specifically intended to  hold confidential data

```Note
Kubernetes Secrets are, by default, stored unencrypted in the API server's underlying data store (etcd). Anyone with API access can retrieve or modify a Secret, and so can anyone with access to etcd. Additionally, anyone who is authorized to create a Pod in a namespace can use that access to read any Secret in that namespace; this includes indirect access such as the ability to create a Deployment.

To make it secure enough : 
TODO: 
```

### Use cases for secrets: 

#### dotfiles in a secret volume 
Define a key that begins with a dot - which represents a dotfile or "hidden" file
For  e.g:  secret -> `secret-volume`  and volume will contain a single file called `.secret-file` at `/etc/secret-volume/.secret-file`


> [!NOTE]- secret/dotfile-secret.yaml
> ```yaml
>apiVersion: v1
>kind: secret
>metadata:
>	name: dotfile-secret
>data:
>	.secret-file: rwrweree=
>---
>apiVersion: v1
>kind: Pod
>metadata:
>	name: secret-dotfiles-pod
>spec:
>	volumes:
>		- name: secret-volume
>		   secret:
>			   secretName: dotfile-secret
>		  
>
> ```
