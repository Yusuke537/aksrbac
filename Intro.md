# Namespaces

Kubernetes supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called namespaces.

## When do they come into play

- When you have environments with a lot of users spread across multiple teams or projects.
- They are used to provide a scope for names. Resources contained in a namespace need to have an unique name. Resource names are independent across namespaces.
- Namespaces can't be nested.
- They provide a neat way to divide cluster resources between users.
- Don't use different namesapces just to separate slightly different resources, ex 2 versions of a same app. That's what labels are for. 

# Users

There are 2 types of users for K8s:
1. K8s managed
2. Normal users

Kubernetes does not have objects which represent normal user accounts. Can't really make API calls to K8s toi add normal users.

## Normal Users

If you can present a valid certificate signed by the cluster's `Certified Authority CA`, K8s is happy with you.

K8s determines the username from the `Common Name` field in the `Certification's subject`.
Then, the `Role Based Access Control RBAC` determines if you should be authorized to perform operations on resources

## Managed users

They are managed by K8s API. 
Will be bound to specific namespaces. 
Either created automatically by the API server or manually through API calls.
They are tied to `Secrets`, which are pod-mounted to allow cluster processes to comunicate witht he API.

## Why is this important?

Every API request is tied to a User, Service Account or Anonymous Request.
Every process in or out-side of the cluster must authenticate when making requests or it will be treated as an anonymous user. 
If not configured, anonymous requests will be rejected.
