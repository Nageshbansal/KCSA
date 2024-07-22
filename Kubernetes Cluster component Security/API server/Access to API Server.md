tags: #APIServer 
references: [Controlling Access](https://kubernetes.io/docs/concepts/security/controlling-access/)

API server is the entry point for interacting with Kubernetes Cluster. Access Management for API server is done through several steps. ( As Show in Below Figure ).

User can Access Kubernetes API server by following methods:
	- Kubectl
	- client libraries
	- REST requests

"Both human users and [Kubernetes service accounts](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/) can be authorized for API access."

![](access-control-overview.svg)


## Components for Access:

### Transport Security

"by default, Kubernetes API Server listens on the port 6443, first non-localhost network interface protected by TLS", In production, it listens on the port "443".
Port can be changed by using the `--secure-port`
Listening IP can be changed by using the `--bind-address`

The API server presents a certificate. This certificate may be signed using a private certificate authority (CA), or based on a public key infrastructure linked to a generally recognized CA. The certificate and corresponding private key can be set by using the `--tls-cert-file` and `--tls-private-key-file` flags.

If your cluster uses a private certificate authority, you need a copy of that CA certificate configured into your `~/.kube/config` on the client, so that you can trust the connection and be confident it was not intercepted.

Your client can present a TLS client certificate at this stage.


### Authentication

ref: [docs](https://kubernetes.io/docs/concepts/security/controlling-access/#authentication)
read more: [Authentication](Authentication.md)

Once TLS is established, request is sent to Auth stage. The Cluster creation script or cluster admin configures the API server to run one or more Authenticator Modules. 

The entire request is sent to these Authenticator Modules, however, they just examines the header, and/or client certificates. Each modules are tried in sequence until one is succedded.

Authentication Modules - client certificates, passwords, plain token, bootstrap token and JSON web tokens (incase of service accounts). 

if the request is not authenticated return `401`, otherewise, the `user` is auntheticated as a specific `username`.  The user name is available to subsequent steps to use in their decisions. Some authenticator  also provide the group memberships of the user, while other authenticators do not.

```
While Kubernetes uses usernames for access control decisions and in request logging, it does not have a `User` object nor does it store usernames or other information about users in its API.
```


### Authorization 

ref:  https://kubernetes.io/docs/concepts/security/controlling-access/#authorization
read more: [Authorization](Authorization.md)

After authenticating a request from a specific user, the request must be authorized.
Request must include:
- username of requester
- the requested action
- object affected by the action

Request is authorized only if an exisiting policy declares that user has access to request resource in the namespace.

An example of the policy:

```yaml
{
	 "apiVersion": "abac.authorization.kubernetes.io/v1beta1",
	 "kind" : "Policy",
	 "spec": {
		 "user": "xyz",
		 "namespace": "abc",
		 "resource": "pods",
		 "readonly": true
	 }
}
```


request:
```yaml
{
	"apiVersion": "authorization.k8s.io/v1beta1",
	"kind": "SubjectAccessReview",
	"spec": {
		"resourcesAttributes": {
			"namespace": "abc",
			"verb": "get",
			"group" : "cxv",
			"resources": "pod",
		}
	}
}
```

Kubernetes Authorization requires that user use common REST attributes to interact with exisiting organization-wide or cluster-wide access control systems. it is important to use REST formatting because these control systems might interact with other APIs besides the Kubernetes API.

Kubernetes provide to use mutltiple authorization modules, such as:
	- ABAC (attribute based access control)
	- RBAC (role based access control)
	- Webhook Mode
When an adminstrator creates a cluster, he configures the modules to be used in Kubernetes API server. if Multiple modules are configured, Kubernetes checks each module, and if any module authorize the request, then the request can proceed. if `all of the modules denies the request` , the request is denied (return HTTP Status code 403).


### Admission Control
ref: 
read more: [Admission Control](Admission%20Control.md)

Admission Controller happens after Authn, Authz. They're software modules which can modify or reject requests. In addition to attributes available to Authn, Authz they can acess the contents of the objects that is being created or modified.

They act on the request that :
- create
- modify
- delete
- connect (proxy) to an object
They dont act on the requests which merely read objects. 
When Multiple admission controllers are configured, they are called in order

```
Unlike Authentication and Authorization modules, if any admission controller module rejects, then the request is immediately rejected.
```

In addition to rejecting objects, admission controllers can also set complex defaults for fields.
Once a request passes all admission controllers, it is validated using the validation routines for the corresponding API object, and then written to the object store


## Auditing[](https://kubernetes.io/docs/concepts/security/controlling-access/#auditing)

Kubernetes auditing provides a security-relevant, chronological set of records documenting the sequence of actions in a cluster. The cluster audits the activities generated by users, by applications that use the Kubernetes API, and by the control plane itself.