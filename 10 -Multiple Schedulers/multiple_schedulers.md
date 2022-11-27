#### Multiple Schedulers

default scheduler configuration file


*scheduler-config.yaml*
```{r echo=FALSE, eval=FALSE}
apiVersion: kubescheduler.config.k8s.io/v1
kind: kubeSchedulerConfiguration
profiles:
  - scheulerName: default-scheduler
```

custom scheduler configuration file

*myscheduler-config.yaml*

```{r echo=FALSE, eval=FALSE}
apiVersion: kubescheduler.config.k8s.io/v1
kind: kubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler

```
#### Deploying default scheduler

- download kube scheduler binary
- Run it as a service with a set of options

```{r echo=FALSE, eval=FALSE}
kubeScheduler.Service
ExecStart=/usr/local/bin/kube-Scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml
```
##### Deploying custom scheduler

**Option 1:**
- download the default scheduler binary
- point configureation to our custom configuration file

```{r echo=FALSE, eval=FALSE}
kubeScheduler.Service
ExecStart=/usr/local/bin/kube-Scheduler \\
  --config=/etc/kubernetes/config/my-scheduler.yaml
````

**Option 2: Deploying scheduler as a pod**

- Create a pod configuration file and specify the kube-config property which is the path to scheduler configuration file that has the authntication information to connect to the kube-apiserver 
- Then pass our custom kube-scheduler config file as a config option to the scheduler

**Leader Elect option:**
- goes into kube-scheduler configuration
- It is used when you have multiple copies of the scheduler running in the master nodes in a HA setup
- The leader elect option chooses the leader scheduler as only one scheduler need to be active at a time

*my-scheduler-config.yaml*

```{r echo=FALSE, eval=FALSE}
apiVersion: kubescheduler.config.k8s.io/v1
kind: kubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
leaderElection:
  leaderElect: true
  resourceNamespace: kube-system 
  resourceName: lock-object-my-scheduler
```


*my-custom-scheduler.yaml*
```{r echo=FALSE, eval=FALSE}
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --config=/etc/kubernetes/myscheduler-config.yaml

    image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
    name: kube-scheduler
```

#### view schedulers by using kubectl get pods --namespace=kube-system


#### Create a pod to use the custom scheduler

```{r echo=FALSE, eval=FALSE}
nginx.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
   schedulerName: my-custom-scheduler
```

 #### View scheduler events

```{r echo=FALSE, eval=FALSE}
 kubectl get events -o wide 
 kubectl logs my-custom-scheduler --namespace=kube-system
```
Some kubernetes objects needed before creating custom scheduler

+ ServiceAccount: my-scheduler (kube-system namespace)
+ ClusterRoleBinding: my-scheduler-as-kube-scheduler
+ ClusterRoleBinding: my-scheduler-as-volume-scheduler

#### Let's create a configmap that the new scheduler will employ using the concept of ConfigMap as a volume.

```bash
controlplane ~ ➜  vi my-scheduler-configmap.yaml
```

```yaml
apiVersion: v1
data:
  my-scheduler-config.yaml: |
    apiVersion: kubescheduler.config.k8s.io/v1beta2
    kind: KubeSchedulerConfiguration
    profiles:
      - schedulerName: my-scheduler
    leaderElection:
      leaderElect: false
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: my-scheduler-config
  namespace: kube-system
```

```bash
controlplane ~ ➜  kubectl apply -f my-scheduler-configmap.yaml
configmap/my-scheduler-config configured
````

Deploy an additional scheduler to the cluster following the given specification.

+ Name: my-scheduler
+ Status: Running
+ Correct image used

```bash
controlplane ~ ➜  vi /root/my-scheduler.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: my-scheduler
  name: my-scheduler
  namespace: kube-system
spec:
  serviceAccountName: my-scheduler
  containers:
  - command:
    - /usr/local/bin/kube-scheduler
    - --config=/etc/kubernetes/my-scheduler/my-scheduler-config.yaml
    image: k8s.gcr.io/kube-scheduler:v1.24.0
    livenessProbe:
      httpGet:
        path: /healthz
        port: 10259
        scheme: HTTPS
      initialDelaySeconds: 15
    name: kube-second-scheduler
    readinessProbe:
      httpGet:
        path: /healthz
        port: 10259
        scheme: HTTPS
    resources:
      requests:
        cpu: '0.1'
    securityContext:
      privileged: false
    volumeMounts:
      - name: config-volume
        mountPath: /etc/kubernetes/my-scheduler
  hostNetwork: false
  hostPID: false
  volumes:
    - name: config-volume
      configMap:
        name: my-scheduler-config
```

#### create a POD with the new custom scheduler

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
  schedulerName: my-scheduler
```

```bash
controlplane ~ ➜  kubectl apply -f nginx-pod.yaml 
pod/nginx created

controlplane ~ ➜  kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          10s
```

***The End***