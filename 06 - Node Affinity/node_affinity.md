### Node Affinity:

#### Method 1: Using Node selectors

#### First apply a label on a node using key-value pair
```bash
controlplane ~ ➜  kubectl label nodes node01 color=blue
node/node01 labeled
```
#### Create a new deployment named blue with the nginx image and 3 replicas.

```bash
controlplane ~ ➜  kubectl create deployment blue --image=nginx --replicas=3 --dry-run=client -o yaml >blue.yaml 

controlplane ~ ➜  
```

#### Next, add the nodeSelector on the pod definition to match the node label

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  nodeSelector:
    size: Large 
```
#### Limitations of method 1:

+ You cannot provide advanced expressions such as OR or NOT with nodeSelectors

#### Method 2: Using Node Affinity. It provides advanced capabilities to limit pod placement on nodes

#### Set Node Affinity to the *blue* deployment to place the pods on node01 only. Property of the deployment should be as follows

```yaml
Name: blue
Replicas: 3
Image: nginx
NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution
Key: color
value: blue

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: blue
  name: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: blue
  template:
    metadata:
      labels:
        app: blue
    spec:
      containers:
      - image: nginx
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                - key: color
                  operator: In
                  values:
                    - blue
```

```bash
controlplane ~ ➜  kubectl apply -f blue.yaml 
deployment.apps/blue configured

controlplane ~ ➜  
```



+ Other Expressions examples:

```bash
;
;
;
nodeAffinity:
      requiredDuringSchedulingIgnoreDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: NotIn
            values:
            - Small

;
;
;
nodeAffinity:
      requiredDuringSchedulingIgnoreDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: Exists
```            

#### Node affinity types

+ Available

requiredDuringSchedulingIgnoredDuringExecution 

```{r echo=FALSE, eval=FALSE}
requiredDuringScheduling - Scheduler will mandate that the pod be placed on a node with the give affinity rules. If it cannot find one, the pod will not be scheduled. Used where pod placement is crucial.
```

preferredDuringSchedulingIgnoredDuringExecution

```{r echo=FALSE, eval=FALSE}
preferredDuringScheduling - Used in the case where the pod placement is not as important as running the workload. Where the matching node is not found, the scheduler will ignore the node affinity rules and place the pod on any available node
```

```{r echo=FALSE, eval=FALSE}
IgnoreDuringExecution - Pods will be continue to run and not be impacted if node lables change after the pods have been scheduled
```

+ Planned (for the future)

requiredDuringSchedulingRequiredDuringExecution

            
```{r echo=FALSE, eval=FALSE}
RequiredDuringExecution - Will evict any pods running on nodes and they do not meet node affinity rules
```


#### Create a new deployment named red with the nginx image and 2 replicas, and ensure it gets placed on the controlplane node only.

#### Use the label key - node-role.kubernetes.io/control-plane - which is already set on the controlplane node.


+ Name: red
+ Replicas: 2
+ Image: nginx
+ NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution
+ Key: node-role.kubernetes.io/control-plane
+ Use the right operator

```bash
controlplane ~ ➜  kubectl create deployment red --image=nginx --replicas=2 --dry-run=client -o yaml >red.yaml 

controlplane ~ ➜  
````

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: red
  name: red
spec:
  replicas: 2
  selector:
    matchLabels:
      app: red
  template:
    metadata:
      labels:
        app: red
    spec:
      containers:
      - image: nginx
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                - key: node-role.kubernetes.io/control-plane
                  operator: Exists
```

```bash

controlplane ~ ➜  kubectl apply -f red.yaml deployment.apps/red created

controlplane ~ ➜ 
``` 