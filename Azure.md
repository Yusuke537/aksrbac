# Azure RBAC

The process for Azure RBAC is pretty much the same as for the normal k8s cluster.
The only difference will be on the processes used and what IDs for which Resources will be required.

## Create Azure Group

1. Get AKS cluster ID
```
AKS_ID=$(az aks show \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --query id -o tsv)
```

2. Create a Group in Azure
```
APPDEV_ID=$(az ad group create --display-name appdev --mail-nickname appdev --query objectId -o tsv)
```

3. Grant Azure Group the role assignment to allow its users to use AKS
```
az role assignment create \
  --assignee $APPDEV_ID \
  --role "Azure Kubernetes Service Cluster User Role" \
  --scope $AKS_ID
```

## Create Users for Azure

4. Create a User that will be part of the Group
```
AKSDEV_ID=$(az ad user create \
  --display-name "AKS Dev" \
  --user-principal-name $AAD_DEV_UPN \
  --password $AAD_DEV_PW \
  --query objectId -o tsv)
```

5. Add the new User to the Group
```
az ad group member add --group appdev --member-id $AKSDEV_ID
```

## Create the cluster resources

6. To set the Role permissions, we need access to admin
```
az ad group member add --group appdev --member-id $AKSDEV_ID
```

7. Create a Role file called role-dev-namespace.yaml
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-full-access
  namespace: dev
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["batch"]
  resources:
  - jobs
  - cronjobs
  verbs: ["*"]
```

8. Apply the newly-created 
```
kubectl apply -f role-dev-namespace.yaml
```

9. Get the Group ID for the role binding
```
az ad group show --group appdev --query objectId -o tsv
```

10. Create a Role Binding for the Group to be used in the Role created for the Namespace and save it as rolebinding-dev-namespace.yaml
```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-access
  namespace: dev
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: dev-user-full-access
subjects:
- kind: Group
  namespace: dev
  name: groupObjectId
```

11. Now that everything is in place, Users should be able to do magic in the cluster based on the presented process
