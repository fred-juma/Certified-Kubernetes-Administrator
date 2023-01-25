#### Network Policy

- By default, kubernetes allow all port traffic from and to all destinations "Allow All"
- We use network policy objects to filter traffic

Below is a network policy that blocks all connections to db pod, except traffic from api-pod destined to db pod on port 3306. This policy applies to the default namespace.


Ingress Network Policy in default namespace             |  
:-------------------------:|
![Network Policy in default namespace  ](https://github.com/fred-juma/Certified-Kubernetes-Administrator/blob/main/images/network-policy-ingress-default-namespace.JPG)


```yaml
apiVersion: networking.k8s.io/v1
kind: networkPolicy
metadata: 
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306
```
Create the policy

kubectl create -f policy-definition.yaml

Create a policy that allows ingress from api-pod in staging namespace to connect to db pod in prod namespace

Ingress Network Policy in non-default namespace             |  
:-------------------------:|
![Network Policy in non-default namespace  ](https://github.com/fred-juma/Certified-Kubernetes-Administrator/blob/main/images/network-policy-ingress-different-namespace.JPG)

```yaml
apiVersion: networking.k8s.io/v1
kind: networkPolicy
metadata: 
  name: db-policy
  namespace: prod
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
      namespaceSelector:
      	matchLabels: 
      		name: staging
    ports:
    - protocol: TCP
      port: 3306
```

Note: if you want all the pods in the staging namespace to access the db pod in the prod namespace, delete the "from -podSelector, matchLabels, name: api-pod"

Configure network traffic to allow traffic originating from certain external ip address to db pod in the prod namespace.

Ingress Network Policy from external ip address            |  
:-------------------------:|
![Network Policy from external ip address](https://github.com/fred-juma/Certified-Kubernetes-Administrator/blob/main/images/network-policy-ingress-external-ipaddress.JPG)

```yaml
apiVersion: networking.k8s.io/v1
kind: networkPolicy
metadata: 
  name: db-policy
  namespace: prod
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
      namespaceSelector:
      	matchLabels: 
      		name: prod
    -ipBlock:
        cidr: 192.168.5.10/32  		
    ports:
    - protocol: TCP
      port: 3306
```

Add an Egress rule for egress traffic originating from the db server in the prod namespace to the external storage server port 80

Network Policy egress rule           |  
:-------------------------:|
![Network Policy egress rule](https://github.com/fred-juma/Certified-Kubernetes-Administrator/blob/main/images/network-policy-egress-rule.JPG)


```yaml
apiVersion: networking.k8s.io/v1
kind: networkPolicy
metadata: 
  name: db-policy
  namespace: prod
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306
  egress: 
  - to:
  	  -ipBlock:
        cidr: 192.168.5.10/32 
      ports:  
      	- protocol: TCP
      	  port: 80
```

- Network policies are enforced by network solutions implemented in the k8s cluster. Supported network solutions include:
- Kube-router
- Calico
- Romana
- Weave-net

- Unspported ones include: Flannel

How many network policies do you see in the environment? **payroll-policy** . The policy is applied on to the payroll pod


We have deployed few web applications, services and network policies. Inspect the environment.

```bash
controlplane ~ ➜  kubectl get networkpolicy
NAME             POD-SELECTOR   AGE
payroll-policy   name=payroll   64s

controlplane ~ ➜ 
```


What type of traffic is this Network Policy configured to handle? **Ingress**. TRaffic from internal pod to payroll pod on TCP port 8080 is allowed.

```bash
controlplane ~ ➜  kubectl describe  networkpolicy payroll-policy
Name:         payroll-policy
Namespace:    default
Created on:   2023-01-24 03:33:59 -0500 EST
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     name=payroll
  Allowing ingress traffic:
    To Port: 8080/TCP
    From:
      PodSelector: name=internal
  Not affecting egress traffic
  Policy Types: Ingress

controlplane ~ ➜  
```

Create a network policy to allow traffic from the Internal application only to the payroll-service and db-service.

Use the spec given below. You might want to enable ingress traffic to the pod to test your rules in the UI.

- Policy Name: internal-policy
- Policy Type: Egress
- Egress Allow: payroll
- Payroll Port: 8080
- Egress Allow: mysql
- MySQL Port: 3306

Network Policy Test         |  
:-------------------------:|
![Network Policy Test ](https://github.com/fred-juma/Certified-Kubernetes-Administrator/blob/main/images/network-policy-quiz.JPG)

View all resources in the cluster's default namespace

```bash
controlplane ~ ➜  kubectl get all
NAME           READY   STATUS    RESTARTS   AGE
pod/external   1/1     Running   0          27m
pod/internal   1/1     Running   0          27m
pod/mysql      1/1     Running   0          27m
pod/payroll    1/1     Running   0          27m

NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/db-service         ClusterIP   10.101.203.49   <none>        3306/TCP         27m
service/external-service   NodePort    10.98.19.93     <none>        8080:30080/TCP   27m
service/internal-service   NodePort    10.97.199.145   <none>        8080:30082/TCP   27m
service/kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP          67m
service/payroll-service    NodePort    10.103.23.167   <none>        8080:30083/TCP   27m

controlplane ~ ➜  
```


View internal pod definition

```yaml
controlplane ~ ➜  kubectl get pod internal -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"name":"internal"},"name":"internal","namespace":"default"},"spec":{"containers":[{"env":[{"name":"APP_NAME","value":"Internal Facing Application"},{"name":"BG_COLOR","value":"blue"}],"image":"kodekloud/webapp-conntest","name":"internal","ports":[{"containerPort":8080}]}]}}
  creationTimestamp: "2023-01-24T08:33:58Z"
  labels:
    name: internal
  name: internal
  namespace: default
  resourceVersion: "3644"
  uid: 9811f8d3-a0b3-4c71-a797-16ca0df7b790
spec:
  containers:
  - env:
    - name: APP_NAME
      value: Internal Facing Application
    - name: BG_COLOR
      value: blue
    image: kodekloud/webapp-conntest
    imagePullPolicy: Always
    name: internal
    ports:
    - containerPort: 8080
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-hkcm7
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
  - name: kube-api-access-hkcm7
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
    lastTransitionTime: "2023-01-24T08:33:58Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2023-01-24T08:34:32Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2023-01-24T08:34:32Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2023-01-24T08:33:58Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://7bad0402871da4c1d4632e5216802e2f5b8477914edd6db3820e8fe459f1e8c1
    image: docker.io/kodekloud/webapp-conntest:latest
    imageID: docker.io/kodekloud/webapp-conntest@sha256:b87d6b99f127ef10274e4447b3a151127688369b82eab09eba6ab6a84e6c5689
    lastState: {}
    name: internal
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2023-01-24T08:34:32Z"
  hostIP: 10.27.96.3
  phase: Running
  podIP: 10.244.0.7
  podIPs:
  - ip: 10.244.0.7
  qosClass: BestEffort
  startTime: "2023-01-24T08:33:58Z"

controlplane ~ ➜  
```
```yaml
apiVersion: networking.k8s.io/v1
kind: networkPolicy
metadata: 
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
  - Egress
  - Ingress
  ingress:
  - {}
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080

  - to:
    - podSelector:
      	matchLabels: 
      		name: mysql
    ports:
    - protocol: TCP
      port: 3306
		
  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP  
```


Note: We have also allowed Egress traffic to TCP and UDP port. This has been added to ensure that the internal DNS resolution works from the internal pod. Remember: The kube-dns service is exposed on port 53:

```bash
root@controlplane:~# kubectl get svc -n kube-system 
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   93m
root@controlplane:~#
```

```bash
controlplane ~ ➜  kubectl create -f internal.yaml 
networkpolicy.networking.k8s.io/internal-policy created

controlplane ~ ➜  
```
