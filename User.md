# User

## Creating a user

1. Generate an RSA key

```
openssl genrsa -out bob.key 4096
```

2. Create a cnf file
- In the CN: you must add the user name.
- In the O: you must add the team/group.
- Save this as csr.cnf CSR=Certificate Signing Request.
```
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
[ dn ]
CN = bob
O = dev
[ v3_ext ]
authorityKeyIdentifier=keyid,issuer:always
basicConstraints=CA:FALSE
keyUsage=keyEncipherment,dataEncipherment
extendedKeyUsage=serverAuth,clientAuth
```

3. Generate a `CSR=Certificate Signing Request` 
```
openssl req -config ./csr.cnf -new -key bob.key -nodes -out bob.csr
```

4. When the CSR is ready, Bob must send it to the admin of the cluster

## Certificate Signing Request

1. Export the generated certificate in base64 value
```
export BASE64_CSR=$(cat ./bob.csr | base64 | tr -d '\n')
```

2. Create a YAML file for the CSR and apply the base64 value
```
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: mycsr
spec:
  groups:
  - system:authenticated
  request: ${BASE64_CSR}
  usages:
  - digital signature
  - key encipherment
  - server auth
  - client auth
```

3. Apply the resource
```
cat csr.yaml | envsubst | kubectl apply -f -
```

4. Check request status
```
kubectl get csr
```

5. Approve the request
```
kubectl certificate approve mycsr
```

6. Building the config for the new user
```
export USER="bob"

export CLUSTER_NAME=$(kubectl config view --minify -o jsonpath={.current-context})

export CLUSTER_ENDPOINT=$(kubectl config view --raw -o json | jq -r '.clusters[] | select(.name == "'$(kubectl config current-context)'") | .cluster."server"')

export CLUSTER_CA=$(kubectl config view --raw -o json | jq -r '.clusters[] | select(.name == "'$(kubectl config current-context)'") | .cluster."certificate-authority-data"')

export CLIENT_CERTIFICATE_DATA=$(kubectl get csr mycsr -o jsonpath='{.status.certificate}')
```

```
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: ${CLUSTER_CA}
    server: ${CLUSTER_ENDPOINT}
  name: ${CLUSTER_NAME}
users:
- name: ${USER}
  user:
    client-certificate-data: ${CLIENT_CERTIFICATE_DATA}
contexts:
- context:
    cluster: ${CLUSTER_NAME}
    user: bob
  name: ${USER}-${CLUSTER_NAME}
current-context: ${USER}-${CLUSTER_NAME}
```

7. Use the generated config for temporary session
```
export KUBECONFIG=$PWD/kubeconfig
```
