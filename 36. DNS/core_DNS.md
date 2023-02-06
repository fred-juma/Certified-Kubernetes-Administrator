FQDN for accessing a service across different namespaces in a cluster:

Hostname: <service_name>
Namespace: <namespace_name>
Type: svc
Root: cluster.local


For example:
- If we have a service called *web-service* in a namespace called *dev*, a pod in a different namespace can access the service by its FQDN:

http://web-service.dev.svc.cluster.local

- This information is maintained by Kube DNS

- Pod information is also maintained by the kube DNS as follows:

Hostname: <pod_ip-address>
Namespace: <namespace_name>
Type: svc
Root: cluster.local

For example:
- If we have a pod whose IP Address is *10.244.1.5* in a namespace called *dev*, a pod in a different namespace can access the pod by its FQDN:

http://10-244-1-5.dev.svc.cluster.local

- Kubernetes deploys a DNS server in the cluster kube-system namespace as a pod, the recommended DNS server is CoreDNS.
- They are deployed for redundancy as part of ReplicaSet ina deployment.
- The pods run ./Coredns executable
- The corefile is located at: */etc/coredns/Corefile*
- Within the Corefile there are plugins configured: errors, health, kybernetes, prometheus, proxy, cache, reload: *kubectl describe configmap coredns -n kube-system*
- Within the kubernetes plugin you set the top level domain name: *cluster.local*
- The corefile is passed to the kubernetes as a configmap:

*kubectl get configmap -n kube-system*

- The coreDNS is made available to the cluster via a service named *kube-dns*

*kubectl get service -n kube-system*

- The IP address of this service is configured as the nameserver on the pods by kubelet when pods are created

*cat /var/lib/kubelet/config.yaml*

Identify the DNS solution implemented in this cluster. **coredns**

```bash
controlplane ~ ➜  kubectl get configmap -n kube-system
NAME                                 DATA   AGE
coredns                              1      2m23s
extension-apiserver-authentication   6      2m29s
kube-proxy                           2      2m23s
kube-root-ca.crt                     1      2m11s
kubeadm-config                       1      2m27s
kubelet-config                       1      2m27s

controlplane ~ ➜  
```

How many pods of the DNS server are deployed? **2**

```bash
controlplane ~ ➜  kubectl get pod -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-787d4945fb-slh7q               1/1     Running   0          3m3s
coredns-787d4945fb-t6kr2               1/1     Running   0          3m3s
etcd-controlplane                      1/1     Running   0          3m13s
kube-apiserver-controlplane            1/1     Running   0          3m13s
kube-controller-manager-controlplane   1/1     Running   0          3m13s
kube-proxy-crznl                       1/1     Running   0          3m3s
kube-scheduler-controlplane            1/1     Running   0          3m16s

controlplane ~ ➜  
```

What is the name of the service created for accessing CoreDNS? **kube-dns**

```bash
controlplane ~ ➜  kubectl get service -n kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   3m57s

controlplane ~ ➜  
```

What is the IP of the CoreDNS server that should be configured on PODs to resolve services? **10.96.0.10**

Where is the configuration file located for configuring the CoreDNS service?

**Args:
      -conf
      /etc/coredns/Corefile**

First check the name of coredns deployment 

```bash
controlplane ~ ➜  kubectl get deployments.apps  -n kube-system
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
coredns   2/2     2            2           7m56s

controlplane ~ ➜  
```

Then check the Args in the deployment

**Args:
      -conf
      /etc/coredns/Corefile**

```bash
controlplane ~ ➜  kubectl -n kube-system describe deployments.apps coredns 
Name:                   coredns
Namespace:              kube-system
CreationTimestamp:      Sat, 04 Feb 2023 06:02:02 -0500
Labels:                 k8s-app=kube-dns
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               k8s-app=kube-dns
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 25% max surge
Pod Template:
  Labels:           k8s-app=kube-dns
  Service Account:  coredns
  Containers:
   coredns:
    Image:       registry.k8s.io/coredns/coredns:v1.9.3
    Ports:       53/UDP, 53/TCP, 9153/TCP
    Host Ports:  0/UDP, 0/TCP, 0/TCP
    Args:
      -conf
      /etc/coredns/Corefile
    Limits:
      memory:  170Mi
    Requests:
      cpu:        100m
      memory:     70Mi
    Liveness:     http-get http://:8080/health delay=60s timeout=5s period=10s #success=1 #failure=5
    Readiness:    http-get http://:8181/ready delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /etc/coredns from config-volume (ro)
  Volumes:
   config-volume:
    Type:               ConfigMap (a volume populated by a ConfigMap)
    Name:               coredns
    Optional:           false
  Priority Class Name:  system-cluster-critical
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   coredns-787d4945fb (2/2 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  9m10s  deployment-controller  Scaled up replica set coredns-787d4945fb to 2

controlplane ~ ➜  
```

How is the Corefile passed into the CoreDNS POD? **it is configured as a ConfigMap object** named **coredns** 

**Volumes:
   config-volume:
    Type:               ConfigMap (a volume populated by a ConfigMap)
    Name:               coredns**

```bash
controlplane ~ ➜  kubectl get configmap -n kube-system
NAME                                 DATA   AGE
coredns                              1      11m
extension-apiserver-authentication   6      11m
kube-proxy                           2      11m
kube-root-ca.crt                     1      11m
kubeadm-config                       1      11m
kubelet-config                       1      11m

controlplane ~ ➜  

```

What is the root domain/zone configured for this kubernetes cluster? **cluster.local**

```bash
controlplane ~ ➜  kubectl describe configmap coredns -n kube-system
Name:         coredns
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Data
====
Corefile:
----
.:53 {
    errors
    health {
       lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf {
       max_concurrent 1000
    }
    cache 30
    loop
    reload
    loadbalance
}


BinaryData
====

Events:  <none>

controlplane ~ ➜  
```

We have deployed a set of PODs and Services in the default and payroll namespaces. Inspect them and go to the next question.

Pods in payroll namespace

```bash
controlplane ~ ➜  kubectl get pod -n payroll
NAME   READY   STATUS    RESTARTS   AGE
web    1/1     Running   0          14m

controlplane ~ ➜  
```

Pods in default namespace

```bash
controlplane ~ ➜  kubectl get pod
NAME                READY   STATUS    RESTARTS   AGE
hr                  1/1     Running   0          15m
simple-webapp-1     1/1     Running   0          14m
simple-webapp-122   1/1     Running   0          14m
test                1/1     Running   0          15m

controlplane ~ ➜  
```

What name can be used to access the hr web server from the test Application? **web-service**


You can execute a curl command on the test pod to test. Alternatively, the test Application also has a UI. Access it using the tab at the top of your terminal named test-app. 

```bash
controlplane ~ ➜  kubectl get svc
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP        24m
test-service   NodePort    10.108.216.171   <none>        80:30080/TCP   22m
web-service    ClusterIP   10.96.179.187    <none>        80/TCP         22m
```

We just deployed a web server - *webapp* - that accesses a database *mysql* - server. However the web server is failing to connect to the database server. Troubleshoot and fix the issue.

They could be in different namespaces. First locate the applications. The web server interface can be seen by clicking the tab Web Server at the top of your terminal.

First check the webapp and mysql pods and which namespace they are deployed in **mysql in payroll namespace, while webapp in default namespace**

```bash
controlplane ~ ➜  kubectl get pod -A
NAMESPACE      NAME                                   READY   STATUS    RESTARTS   AGE
default        hr                                     1/1     Running   0          29m
default        simple-webapp-1                        1/1     Running   0          28m
default        simple-webapp-122                      1/1     Running   0          28m
default        test                                   1/1     Running   0          29m
default        webapp-578c778d58-blrrm                1/1     Running   0          85s
kube-flannel   kube-flannel-ds-sq879                  1/1     Running   0          30m
kube-system    coredns-787d4945fb-slh7q               1/1     Running   0          30m
kube-system    coredns-787d4945fb-t6kr2               1/1     Running   0          30m
kube-system    etcd-controlplane                      1/1     Running   0          30m
kube-system    kube-apiserver-controlplane            1/1     Running   0          30m
kube-system    kube-controller-manager-controlplane   1/1     Running   0          30m
kube-system    kube-proxy-crznl                       1/1     Running   0          30m
kube-system    kube-scheduler-controlplane            1/1     Running   0          30m
payroll        mysql                                  1/1     Running   0          85s
payroll        web                                    1/1     Running   0          29m

controlplane ~ ➜  
```

Edit the web environment variable to include the namespace of myqsl pod 


```bash

controlplane ~ ➜  kubectl edit deploy webapp

spec:
      containers:
      - env:
        - name: DB_Host
          value: mysql.payroll
        - name: DB_User
          value: root
        - name: DB_Password
          value: paswrd

```

From the hr pod nslookup the mysql service and redirect the output to a file /root/CKA/nslookup.out

```bash
controlplane ~ ➜  kubectl exec -it hr -- nslookup mysql.payroll > /root/CKA/nslookup.out



controlplane ~ ➜  cat /root/CKA/nslookup.out 
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   mysql.payroll.svc.cluster.local
Address: 10.97.84.34


controlplane ~ ➜  
```

***The End***