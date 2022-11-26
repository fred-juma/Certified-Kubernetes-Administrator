### Resource REquirements and Limits 

#### Resource Requests - The minimmum amount of CPU and memory assigned to a container. Default values are:
+ 0.5 cpu
+ 256 Mi of memory

#### For the POD to pick up those defaults you must have first set those as default values for request and limit by creating a *LimitRange* in that namespace for memory and cpu. 

mem-limit-range.yaml 

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```

cpu-limit-range.yaml

```bash
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
```

#### Modifying the values in pod definition file 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
    ports:
    - containerPort: 8080
    resources:
      requests:
        memory: "1Gi"
        cpu: 1
 ```

####  Memory units:
+ 1G (Gigabyte) = 1,000,000,000 bytes
+ 1M (Megabyte) = 1,000,000 bytes
+ 1K (Kilobyte) = 1,000 bytes


+ 1Gi (Gibibyte) = 1,073,741,824 bytes
+ 1Mi (Mebibyte) = 1,048,576 bytes
+ 1Ki (Kibibyte) = 1,024 bytes

#### Resource Limits - The maximum values of CPU amd memory that a container can consume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
    ports:
    - containerPort: 8080
    resources:
      requests:
        memory: "1Gi"
        cpu: 1
      limits:
        memory: "2Gi"
        cpu: 2
```
#### Kubernetes behavior when a pod resource requests exceed limits:

+ CPU request exceeding the limits is throttled
+ Memory request exeeding the limits leads to pod termination

```bash
controlplane ~ ➜  kubectl get pods -o wide
NAME       READY   STATUS      RESTARTS      AGE    IP           NODE           NOMINATED NODE   READINESS GATES
elephant   0/1     OOMKilled   4 (60s ago)   108s   10.42.0.10   controlplane   <none>           <none>
```

The status OOMKilled indicates that it is failing because the pod ran out of memory. Identify the memory limit set on the POD.

####  extract the pod definition in YAML format to a file using the command

```bash
kubectl get pods elephant -o yaml > elephant.yaml
```
Then recrete the pod and force deletion of old pod

```bash
kubectl replace --force -f elephant.yaml
```