#### Service Accounts

- Used by an application to interract with the cluster


View service account

```bash

controlplane ~ ➜  kubectl get serviceaccount
NAME      SECRETS   AGE
default   0         6m10s
dev       0         28s

controlplane ~ ➜  
```
What is the secret token used by the default service account?

kubectl describe serviceaccount <name>

```bash
controlplane ~ ➜  kubectl describe serviceaccount default 
Name:                default
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   <none>
Tokens:              <none>
Events:              <none>

controlplane ~ ➜  
```

We just deployed the Dashboard application. Inspect the deployment. What is the image used by the deployment?

```bash
controlplane ~ ➜  kubectl get deployment -o wide
NAME            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS      IMAGES                                                 SELECTOR
web-dashboard   1/1     1            1           46s   web-dashboard   gcr.io/kodekloud/customimage/my-kubernetes-dashboard   name=web-dashboard

controlplane ~ ➜  
```
- When service account is create, a token is also created alongside with it. The token is what is used by the application when authenticating to the cluster
- The token is stored as a secret object
- The secret object is linked to the service account by its name


Inspect the Dashboard Application POD and identify the Service Account mounted on it. **serviceAccountName: default**

First view the pod name

```bash
controlplane ~ ➜  kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
web-dashboard-65b9cf6cbb-6v9zl   1/1     Running   0          3m17s

controlplane ~ ➜  
```

Then describe the pod

```bash

ccontrolplane ~ ➜  kubectl get pod web-dashboard-65b9cf6cbb-6v9zl -o yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: Pod
  metadata:
    creationTimestamp: "2023-01-23T07:15:43Z"
    generateName: web-dashboard-65b9cf6cbb-
    labels:
      name: web-dashboard
      pod-template-hash: 65b9cf6cbb
    name: web-dashboard-65b9cf6cbb-6v9zl
    namespace: default
    ownerReferences:
    - apiVersion: apps/v1
      blockOwnerDeletion: true
      controller: true
      kind: ReplicaSet
      name: web-dashboard-65b9cf6cbb
      uid: d792cad3-9f5d-4611-8ec6-1f11eea5643f
    resourceVersion: "771"
    uid: 671bc3d5-168e-4559-ad98-0773748b734b
  spec:
    containers:
    - env:
      - name: PYTHONUNBUFFERED
        value: "1"
      image: gcr.io/kodekloud/customimage/my-kubernetes-dashboard
      imagePullPolicy: Always
      name: web-dashboard
      ports:
      - containerPort: 8080
        protocol: TCP
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      volumeMounts:
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: kube-api-access-w25f8
        readOnly: true
    dnsPolicy: ClusterFirst
    enableServiceLinks: true
    nodeName: controlplane
    preemptionPolicy: PreemptLowerPriority
    priority: 0
    restartPolicy: Always
    schedulerName: default-scheduler
    securityContext: {}
    serviceAccount: default
    serviceAccountName: default
    terminationGracePeriodSeconds: 30
    tolerations:
    - effect: NoExecute
      key: node.kubernetes.io/not-ready
      operator: Exists
      tolerationSeconds: 300
    - effect: NoExecute
      key: node.kubernetes.io/unreachable
      operator: Exists
      tolerationSeconds: 300
    volumes:
    - name: kube-api-access-w25f8
      projected:
        defaultMode: 420
        sources:
        - serviceAccountToken:
            expirationSeconds: 3607
            path: token
        - configMap:
            items:
            - key: ca.crt
              path: ca.crt
            name: kube-root-ca.crt
        - downwardAPI:
            items:
            - fieldRef:
                apiVersion: v1
                fieldPath: metadata.namespace
              path: namespace
  status:
    conditions:
    - lastProbeTime: null
      lastTransitionTime: "2023-01-23T07:15:43Z"
      status: "True"
      type: Initialized
    - lastProbeTime: null
      lastTransitionTime: "2023-01-23T07:15:49Z"
      status: "True"
      type: Ready
    - lastProbeTime: null
      lastTransitionTime: "2023-01-23T07:15:49Z"
      status: "True"
      type: ContainersReady
    - lastProbeTime: null
      lastTransitionTime: "2023-01-23T07:15:43Z"
      status: "True"
      type: PodScheduled
    containerStatuses:
    - containerID: containerd://bcd60425e4e502b2ecd19628dbb60163a22d90bd95f9e30e98cb9d924aabd84f
      image: gcr.io/kodekloud/customimage/my-kubernetes-dashboard:latest
      imageID: gcr.io/kodekloud/customimage/my-kubernetes-dashboard@sha256:7d70abe342b13ff1c4242dc83271ad73e4eedb04e2be0dd30ae7ac8852193069
      lastState: {}
      name: web-dashboard
      ready: true
      restartCount: 0
      started: true
      state:
        running:
          startedAt: "2023-01-23T07:15:49Z"
    hostIP: 172.25.1.23
    phase: Running
    podIP: 10.42.0.9
    podIPs:
    - ip: 10.42.0.9
    qosClass: BestEffort
    startTime: "2023-01-23T07:15:43Z"
kind: List
metadata:
  resourceVersion: ""

controlplane ~ ➜  
```

At what location is the ServiceAccount credentials available within the pod? **- mountPath: /var/run/secrets/kubernetes.io/serviceaccount**


The application needs a ServiceAccount with the Right permissions to be created to authenticate to Kubernetes. The default ServiceAccount has limited access. Create a new ServiceAccount named dashboard-sa.



```bash
controlplane ~ ➜  kubectl create serviceaccount dashboard-sa
serviceaccount/dashboard-sa created

controlplane ~ ➜  
```

We just added additional permissions for the newly created dashboard-sa account using RBAC. If you are interested checkout the files used to configure RBAC at /var/rbac. We will discuss RBAC in a separate section.

The role

```yaml
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups:
  - ''
  resources:
  - pods
  verbs:
  - get
  - watch
  - list
  ```

Role Binding

```yaml
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: dashboard-sa # Name is case sensitive
  namespace: default
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

Create an authorization token for the newly created service account, copy the generated token and paste it into the token field of the UI.

To do this, run kubectl create token dashboard-sa for the dashboard-sa service account, copy the token and paste it in the UI.

```bash
controlplane /var/rbac ➜  kubectl create token dashboard-sa
eyJhbGciOiJSUzI1NiIsImtpZCI6ImktRUFCSDRaX24tbFl3YzZlaTRWOWJZYUFkOGVYUVNqNThFN09vMXdtTWMifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiLCJrM3MiXSwiZXhwIjoxNjc0NDYzMDMyLCJpYXQiOjE2NzQ0NTk0MzIsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0Iiwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImRhc2hib2FyZC1zYSIsInVpZCI6ImYzMWJiNzlhLTgzYjItNGNkMi1iN2E4LTYxZDlhN2Q4YTczNSJ9fSwibmJmIjoxNjc0NDU5NDMyLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkYXNoYm9hcmQtc2EifQ.P2sotNhQOLgj0JFDB3zATnVXyokao9Mu7csUIeTtOTAtveu8zNzrfQJxG7J-90gWrkzxl9FaN8-OICNlZxvfbWOEbEguFkTY-axf633oKL7tjgpNVgjgPh2C8STB7MpdVl3sz1_uHStJspwuh4A6aDsWNtznkPwawWiIJF4wBNcTCQm4G9MhYVOFo7mfJFJM9gnHLtGwb19YU2cTdFdDvWI7YdmXlGYy3zJpF-TcAJNAgZIc_H4tPjvJ-V6nUzwwuOjPe57LhrWYgsU5DNxg7bvjiAXFCk9Od_7sWYd_uzpwOKMFArpxWAABplACIaQwQqHPqBO-3ACErgQjDpqnNQ

controlplane /var/rbac ➜ 
```

Dashboard Token              |  
:-------------------------:|
![dashboard token](https://github.com/fred-juma/Certified-Kubernetes-Administrator/blob/main/images/service-account-token.JPG)


You shouldn't have to copy and paste the token each time. The Dashboard application is programmed to read token from the secret mount location. However currently, the default service account is mounted. Update the deployment to use the newly created ServiceAccount

Edit the deployment to change ServiceAccount from default to dashboard-sa

First note the deployment name

```bash
controlplane /var/rbac ➜  kubectl get deployment
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
web-dashboard   1/1     1            1           25m

controlplane /var/rbac ➜  
```

Set the service account
controlplane / ➜  kubectl set serviceaccount deploy/web-dashboard dashboard-sadeployment.apps/web-dashboard serviceaccount updated

controlplane / ➜  

View the yaml file

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-dashboard
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      name: web-dashboard
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        name: web-dashboard
    spec:
      serviceAccountName: dashboard-sa
      containers:
      - image: gcr.io/kodekloud/customimage/my-kubernetes-dashboard
        imagePullPolicy: Always
        name: web-dashboard
        ports:
        - containerPort: 8080
          protocol: TCP
```

View the secret

kubectl describe secret <token_name>

- The token can then be used as an authentication bearer token  when making a REST call to the API

curl https://<ip_address>/api -insecure --header "Authorization: Bearer <secret>"

- You can sign permissions to token using role based mechanisms and export the service account token to use it in your application to authenticate to the kubernetes api.
- if the application is hosted in the k8s pod, the above process can be simplified by mounting the service token secret as a volume inside the pod hosting the application 
- Each k8s namespace has its "default" service account. The token are then automatically mounted to pods created in the namespace as volume mounts
- If you dont want the serviceaccount token mounted by default, specify it in pod definitino file:
....
spec:
  automountServiceAccountToken: false


to decode a token

jq -R 'split(".") | select(length > 0) | .[0], .[1] | @base64d | fromjson' <<< <secret>
	

***The End***