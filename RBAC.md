# Role Based Access Control

To get an idea on how those will interact, the model is pretty simple for the RBAC.

Roles -- RoleBinding -- User || Resource

## Namespace

1. Create a Namespace for testing
```
kubectl create ns development
```

## Role

2. Create a role and save it in a file `role.yaml`

In here you can see the roles that will be applied.

The first `apiGroups` represents the `Pods & Services`, which belongs to the `core API group`.
The second `apiGroups` is the `Deployments`, which belongs to the `apps` group.

Each rule will have an `apiGroup`, a set of `resources` and a set of `verbs` that define the allowed actions

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
 namespace: development
 name: dev
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["*"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["create", "get", "update", "list", "delete"]
```

4. Apply the role
```
kubectl apply -f role.yaml
```

5. Create a role binding to a user and save it in a file `binding.yaml`

This just binds the `Role` to an existing `User`. Otherwise Bob won't be ableto perform what is specified in the role.

Can this be done more efficiently? Yes, we can always link the `Role` to the `Group` that Bob is part of, by changing the `Subjects`

```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
 name: dev
 namespace: development
subjects:
- kind: User
  name: bob
  apiGroup: rbac.authorization.k8s.io
roleRef:
 kind: Role
 name: dev
 apiGroup: rbac.authorization.k8s.io
```

6. Apply the Role Binding
```
kubectl apply -f binding.yaml
```

## Observations
There is also something called a `ClusterRole`. This cluster role is smiliar to what was shown above. The difference is that a cluster role does not link to a namespace.

Why?

A namespace is a virtual cluster that's part of the physical cluster.

A `Cluster Role` has power over the whole cluster.
