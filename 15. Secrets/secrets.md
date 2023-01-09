#### Secrets 

- used to store sensitive information like passwords or keys.

Similar to configMaps but stored in encoded formats.

Like Config maps secrets are created then injected into pods

Creating secrets:

1. Imperative method


```bash

kubectl create secret generic \

<secret-name> --from-literal=<key>=<value>

```
Example:

```bash

kubectl create secret generic \

app-secret --from-literal=DB_Host=mysql
           --from-literal=DB_User=root
           --from-literal=DB_Password=passwd  
```

From file:

```bash

kubectl create secret generic \

app-secret --from-file=app_secret.properties
```


2. Declarative way

Create a definition file - secret-data.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: bXlzcWw=
  DB_User: cm9vdA==
  DB_Password: cGFzc3dk
```
The encoded values are derived in the bash as follows:

```bash
$ echo -n 'mysql' | base64
bXlzcWw=
```

```bash
$ echo -n 'root' | base64
cm9vdA==
```

```bash
$ echo -n 'passwd' | base64
cGFzc3dk
```

View secrets

```bash
controlplane ~ ➜  kubectl get secrets
NAME              TYPE                                  DATA   AGE
dashboard-token   kubernetes.io/service-account-token   3      29s

controlplane ~ ➜  
```

To show secrets attributes

```bash
controlplane ~ ➜  kubectl describe secrets dashboard-token
Name:         dashboard-token
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dashboard-sa
              kubernetes.io/service-account.uid: 0260a413-29dd-4f2f-8d9e-2d996d0a5ec9

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     570 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IlZ2N1BVMUwyQWNzYU1OSFVMZW1CQXdROHBqOG9nVkpNcWRaWE9Eb2xJQ2cifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRhc2hib2FyZC10b2tlbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJkYXNoYm9hcmQtc2EiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwMjYwYTQxMy0yOWRkLTRmMmYtOGQ5ZS0yZDk5NmQwYTVlYzkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkYXNoYm9hcmQtc2EifQ.Si9BFFUU5n96uq1VmV0Ww36RIUi1DpMNrjTxUaiB_AKnpWykMoQE8vJEgzXtQCu2NLAtRCIz5cU4yp0dvwazKi66FFnY7AeLNp4Pbk9oDwsptopsZDpRSqFcY8PzZatdK43qUH4sZZAz8lFStxu3-0j5oz9LaUQYQmrAZfOLWJINMvhJrmkj7cEO6NGJjwjo8rmQ-kXRD9ljr4IioEsegokJQF2Vz45AVlWl1iqF5kkcKCZOWBsPfko28i5s6Ni-v0h2xcwAGUNHnpQx9ub8yT3vZOyawVO6RW-04lBIzXLfKByYkM3e1_YvJ7JxOeUF1m-NrVNa9oju20g58ym67g

controlplane ~ ➜  
```

To see encoded values of secrets

```bash

controlplane ~ ➜  kubectl get secrets dashboard-token -o yaml
apiVersion: v1
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJkekNDQVIyZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQWpNU0V3SHdZRFZRUUREQmhyTTNNdGMyVnkKZG1WeUxXTmhRREUyTnpNeU5qTXpOREV3SGhjTk1qTXdNVEE1TVRFeU1qSXhXaGNOTXpNd01UQTJNVEV5TWpJeApXakFqTVNFd0h3WURWUVFEREJock0zTXRjMlZ5ZG1WeUxXTmhRREUyTnpNeU5qTXpOREV3V1RBVEJnY3Foa2pPClBRSUJCZ2dxaGtqT1BRTUJCd05DQUFSNEd4K1YvR2FUVFRjbE00UGtSY3hGTHZueHRVUGx4RTZtWU8rYmN4dnEKc3VCRmhUT0xleEZCb3RlVEg2Z2duVm51WHJTd29SNTNaMVpFaUd1QWVPazhvMEl3UURBT0JnTlZIUThCQWY4RQpCQU1DQXFRd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZEJnTlZIUTRFRmdRVVJQZ0pncTJwSEExWHBFYzV6ZWpPCjhvR0xsZUV3Q2dZSUtvWkl6ajBFQXdJRFNBQXdSUUloQUpzME1oUG9OVUV5TnJWdGd3RE93RFVwdEI0SWRZaHcKZDdXV1RuTjNsN1BWQWlCS29qemV4aXdONGlWRmlud1ZRS2Jabnp6NUpNZnAzdFc4VzM5ekIydytHQT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  namespace: ZGVmYXVsdA==
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklsWjJOMUJWTVV3eVFXTnpZVTFPU0ZWTVpXMUNRWGRST0hCcU9HOW5Wa3BOY1dSYVdFOUViMnhKUTJjaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUprWldaaGRXeDBJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpXTnlaWFF1Ym1GdFpTSTZJbVJoYzJoaWIyRnlaQzEwYjJ0bGJpSXNJbXQxWW1WeWJtVjBaWE11YVc4dmMyVnlkbWxqWldGalkyOTFiblF2YzJWeWRtbGpaUzFoWTJOdmRXNTBMbTVoYldVaU9pSmtZWE5vWW05aGNtUXRjMkVpTENKcmRXSmxjbTVsZEdWekxtbHZMM05sY25acFkyVmhZMk52ZFc1MEwzTmxjblpwWTJVdFlXTmpiM1Z1ZEM1MWFXUWlPaUl3TWpZd1lUUXhNeTB5T1dSa0xUUm1NbVl0T0dRNVpTMHlaRGs1Tm1Rd1lUVmxZemtpTENKemRXSWlPaUp6ZVhOMFpXMDZjMlZ5ZG1salpXRmpZMjkxYm5RNlpHVm1ZWFZzZERwa1lYTm9ZbTloY21RdGMyRWlmUS5TaTlCRkZVVTVuOTZ1cTFWbVYwV3czNlJJVWkxRHBNTnJqVHhVYWlCX0FLbnBXeWtNb1FFOHZKRWd6WHRRQ3UyTkxBdFJDSXo1Y1U0eXAwZHZ3YXpLaTY2RkZuWTdBZUxOcDRQYms5b0R3c3B0b3BzWkRwUlNxRmNZOFB6WmF0ZEs0M3FVSDRzWlpBejhsRlN0eHUzLTBqNW96OUxhVVFZUW1yQVpmT0xXSklOTXZoSnJta2o3Y0VPNk5HSmp3am84cm1RLWtYUkQ5bGpyNElpb0VzZWdva0pRRjJWejQ1QVZsV2wxaXFGNWtrY0tDWk9XQnNQZmtvMjhpNXM2TmktdjBoMnhjd0FHVU5IbnBReDl1Yjh5VDN2Wk95YXdWTzZSVy0wNGxCSXpYTGZLQnlZa00zZTFfWXZKN0p4T2VVRjFtLU5yVk5hOW9qdTIwZzU4eW02N2c=
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Secret","metadata":{"annotations":{"kubernetes.io/service-account.name":"dashboard-sa"},"name":"dashboard-token","namespace":"default"},"type":"kubernetes.io/service-account-token"}
    kubernetes.io/service-account.name: dashboard-sa
    kubernetes.io/service-account.uid: 0260a413-29dd-4f2f-8d9e-2d996d0a5ec9
  creationTimestamp: "2023-01-09T11:26:08Z"
  name: dashboard-token
  namespace: default
  resourceVersion: "685"
  uid: ce33523b-24b4-472d-b1be-e33cb9468bb8
type: kubernetes.io/service-account-token

controlplane ~ ➜  
```

Decode encoded values

```bash
$ echo -n 'bXlzcWw=' | base64 --decode
mysql
```

```bash
$ echo -n 'cm9vdA==' | base64 --decode
root
```

```bash
$ echo -n 'cGFzc3dk' | base64 --decode
passwd
```
Configuring Secrets with a pod

pod-definition.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    envFrom:
      - secretRef:
          name: app-secret
```

We have already deployed the required pods and services. Check out the pods and services created.

Secrets lab Architecture              |  
:-------------------------:|
![deployment](images/secrets-deployment.jpeg)

```bash
controlplane ~ ➜  kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
webapp-pod   1/1     Running   0          85s
mysql        1/1     Running   0          85s

controlplane ~ ➜  kubectl get all
NAME             READY   STATUS    RESTARTS   AGE
pod/webapp-pod   1/1     Running   0          94s
pod/mysql        1/1     Running   0          94s

NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/kubernetes       ClusterIP   10.43.0.1      <none>        443/TCP          27m
service/webapp-service   NodePort    10.43.99.231   <none>        8080:30080/TCP   94s
service/sql01            ClusterIP   10.43.159.8    <none>        3306/TCP         94s

controlplane ~ ➜  
```

Create a new secret named db-secret with the data given below.

You may follow any one of the methods discussed in lecture to create the secret.


Secret Name: db-secret

Secret 1: DB_Host=sql01

Secret 2: DB_User=root

Secret 3: DB_Password=password123

First create the encoded values for the secrets

```bash
controlplane ~ ➜  echo -n 'sql01' | base64
c3FsMDE=
```

```bash
controlplane ~ ➜  echo -n 'root' | base64
cm9vdA==
```

```bash
controlplane ~ ➜  echo -n 'password123' | base64
cGFzc3dvcmQxMjM=

controlplane ~ ➜  
```

Next create the secrets manifest file

```bash

controlplane ~ ➜  vi db-secret.yaml

controlplane ~ ➜  
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
data:
  DB_Host: c3FsMDE=
  DB_User: cm9vdA==
  DB_Password: cGFzc3dvcmQxMjM=
```

Apply the manifest to create the secret resource

```bash
controlplane ~ ➜  kubectl apply -f db-secret.yaml 
secret/db-secret created

controlplane ~ ➜  
```

Configure webapp-pod to load environment variables from the newly created secret.

Delete and recreate the pod if required.


Pod name: webapp-pod

Image name: kodekloud/simple-webapp-mysql

Env From: Secret=db-secret

```bash
controlplane ~ ➜  kubectl edit pod webapp-pod
error: pods "webapp-pod" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-327425250.yaml"
error: Edit cancelled, no valid changes were saved.


apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-pod
  name: webapp-pod
  namespace: default
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    envFrom:
      - secretRef:
          name: db-secret
```

Delete the pod

```bash
controlplane ~ ➜  kubectl delete pod webapp-pod
pod "webapp-pod" deleted

controlplane ~ ➜  
```

Recreate the pod

```bash
controlplane ~ ➜  kubectl apply -f /tmp/kubectl-edit-327425250.yaml
pod/webapp-pod created

controlplane ~ ➜  
```

Notes:
- Secrtes are not encrypted, they are only encoded
- Do not check-in Secret objects to SCM
- Secrets are not encrypted in ETCD. Consider encrypting secret data at rest
- Anyone able to create pods/deployments in the same namespace can access the secrets
- Configure least-privilege access to Secrets - RBAC
- Consider third-party secrets store providers: AWS Provider, Azure Provider, GCP Provider, Vault Provider

***The End***