#### jsonpath in kubectl

Process:
- Identify the *kubectl* command
- Output the result in *JSON* format
- Form the *JSON PATH* query ('$' is not mandatory, kubectl adds it)
- use the *JSON PATH* query with *kubectl* command
Example

kubectl get pods -o=jsonpath='{ .items[0].spec.containers[0].image}'

Validate the same online https://jsonpath.com/

Example: Get nodes and their respective cpu capacity

```bash
kubectl get pods -o=jsonpath='{ .items[*].metadata.name}{"\n"}{ .items[*].status.capacity.cpu}'
```
*Output:*

master node01
4        2


To format the result so that 1st column is nodes and 2nd column is cpu, use the for loop: 

Algorithm:

FOR EACH NODE
  PRINT NODE NAME \t PRINT CPU COUNT \n
END FOR

syntax:

'{range.items[*]}
  { .metadata.name}{"\t"}{ .status.capacity.cpu}{"\n"}
{end}'



```bash
kubectl get pods -o=jsonpath= \
'{ range.items[*].metadata.name}{"\t"}{ .status.capacity.cpu}{"\n"}{end}'
```
*Output:* 

master    4
node01    2

Using custom columns instead of loops.

```bash
kubectl get pods -o=custom-columns=NODE:.metadata.name,CPU.status.capacity.cpu
```

*Output:* 

NODE     CPU
master    4
node01    2

JSOP PATH for sort

kubectl get nodes --sort-by=.metadata.name

kubectl get nodes --sort-by=.status.capacity.cpu

##### 1 / 14 JSON PATH in Kubernetes
Welcome! Let's start slow and we will begin with easy viewing exercises and then move on to building JSON PATH queries.

In the given data, what is the data type of the root element? - **Dictionary**

**Source Data:**

```json
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "nginx-pod",
    "namespace": "default"
  },
  "spec": {
    "containers": [
      {
        "image": "nginx:alpine",
        "name": "nginx"
      }
    ],
    "nodeName": "node01"
  }
}
```

##### 2 / 14 JSON PATH in Kubernetes
How many properties/fields does the root object(dictionary) have? - **apiVersion, kind, metadata and spec**


##### 3 / 14 JSON PATH in Kubernetes
What is the data type of the apiVersion field? - **string**

##### 4 / 14 JSON PATH in Kubernetes
What is the data type of the metadata field? - **Dictionary**

##### 5 / 14 JSON PATH in Kubernetes
Which of the below is the best description of the type of data in the containers field? - **List of dictionaries**

##### 6 / 14 JSON PATH in Kubernetes
Now let's get into some action with JSON PATH. Develop a JSON PATH query to extract the kind of the object


**Expected Output:**

```json
[
  "Pod"
]
```

**JSON Path query:**
```bash
$.kind
```

##### 7 / 14 JSON PATH in Kubernetes
Develop a JSON PATH query to get the name of the POD .

**Expected Output:**

```json
[
  "nginx-pod"
]
```

**JSON Path query:**
```bash
$.metadata.name
```

##### 8 / 14 JSON PATH in Kubernetes
Develop a JSON PATH query to get the node name under the spec section.

**Expected Output:**

```json
[
  "node01"
]
```

**JSON Path query:**
```bash
$.spec.nodeName
```

##### 9 / 14 JSON PATH in Kubernetes
Develop a JSON PATH query to get the details of the container.

**Expected Output:**

```json
[
  {
    "image": "nginx:alpine",
    "name": "nginx"
  }
]
```

**JSON Path query:**
```bash
$.spec.containers[:]
```

##### 10 / 14 JSON PATH in Kubernetes
Develop a JSON PATH query to get the image name under the containers section.

**Expected Output:**

```json
[
  "nginx:alpine"
]
```

**JSON Path query:**
```bash
$.spec.containers[:].image
```

##### 11 / 14 JSON PATH in Kubernetes
We now have POD status too. Develop a JSON PATH query to get the phase of the pod under the status section.

**Source Data:**

```json
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "nginx-pod",
    "namespace": "default"
  },
  "spec": {
    "containers": [
      {
        "image": "nginx:alpine",
        "name": "nginx"
      },
      {
        "image": "redis:alpine",
        "name": "redis-container"
      }
    ],
    "nodeName": "node01"
  },
  "status": {
    "conditions": [
      {
        "lastProbeTime": null,
        "lastTransitionTime": "2019-06-13T05:34:09Z",
        "status": "True",
        "type": "Initialized"
      },
      {
        "lastProbeTime": null,
        "lastTransitionTime": "2019-06-13T05:34:09Z",
        "status": "True",
        "type": "PodScheduled"
      }
    ],
    "containerStatuses": [
      {
        "image": "nginx:alpine",
        "name": "nginx",
        "ready": false,
        "restartCount": 4,
        "state": {
          "waiting": {
            "reason": "ContainerCreating"
          }
        }
      },
      {
        "image": "redis:alpine",
        "name": "redis-container",
        "ready": false,
        "restartCount": 2,
        "state": {
          "waiting": {
            "reason": "ContainerCreating"
          }
        }
      }
    ],
    "hostIP": "172.17.0.75",
    "phase": "Pending",
    "qosClass": "BestEffort",
    "startTime": "2019-06-13T05:34:09Z"
  }
}
```

**Expected Output:**

```json
[
  "Pending"
]
```

**JSON Path query:**
```bash
$.status.phase
```

##### 12 / 14 JSON PATH in Kubernetes
Develop a JSON PATH query to get the reason for the state of the container under the status section.



**Expected Output:**

```json
[
  "ContainerCreating"
]
```

**JSON Path query:**
```bash
$.status.containerStatuses.[0].state.waiting.reason
```

##### 13 / 14 JSON PATH in Kubernetes
Develop a JSON PATH query to get the restart count of redis-container under the status section.

**Expected Output:**

```json
[
  2
]
```

**JSON Path query:**
```bash
$.status.containerStatuses.[1].restartCount
```

##### 14 / 14 JSON PATH in Kubernetes
The order of container could be different. The redis-container need not be the second container all the time. So, re-develop a JSON PATH query to get the restart count of redis-container, but this time use a criteria/condition to get the restart count of the container with the container name redis-container .

**Expected Output:**

```json
[
  2
]
```

**JSON Path query:**
```bash
$.status.containerStatuses[?(@.name == 'redis-container')].restartCount
```

##### 1 / 4 JSON PATH in Kubernetes
Welcome! Let's start slow and we will begin with easy viewing exercises and then move on to building JSON PATH queries.

In the given data, what is the data type of the root element? - **List**

**Source Data:**

```json
[
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "web-pod-1",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "nginx:alpine",
          "name": "nginx"
        }
      ],
      "nodeName": "node01"
    }
  },
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "web-pod-2",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "nginx:alpine",
          "name": "nginx"
        }
      ],
      "nodeName": "node02"
    }
  },
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "web-pod-3",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "nginx:alpine",
          "name": "nginx"
        }
      ],
      "nodeName": "node01"
    }
  },
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "web-pod-4",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "nginx:alpine",
          "name": "nginx"
        }
      ],
      "nodeName": "node01"
    }
  },
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "db-pod-1",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "mysql",
          "name": "mysql"
        }
      ],
      "nodeName": "node01"
    }
  }
]
```

##### 2 / 4 JSON PATH in Kubernetes
How many items do the list have? **5 Dictionaries**

##### 3 / 4 JSON PATH in Kubernetes
Develop a JSON PATH query to get all pod names.

**Expected Output:**

```json
[
  "web-pod-1",
  "web-pod-2",
  "web-pod-3",
  "web-pod-4",
  "db-pod-1"
]
```

**JSON Path query:**
```bash
$.[]metadata.name
```

##### 4 / 4 JSON PATH in Kubernetes
Develop a JSON PATH query to get all user names.

**Source Data:**

```json
{
  "kind": "Config",
  "apiVersion": "v1",
  "preferences": {},
  "clusters": [
    {
      "name": "development",
      "cluster": {
        "server": "KUBE_ADDRESS",
        "certificate-authority": "/etc/kubernetes/pki/ca.crt"
      }
    },
    {
      "name": "kubernetes-on-aws",
      "cluster": {
        "server": "KUBE_ADDRESS",
        "certificate-authority": "/etc/kubernetes/pki/ca.crt"
      }
    },
    {
      "name": "production",
      "cluster": {
        "server": "KUBE_ADDRESS",
        "certificate-authority": "/etc/kubernetes/pki/ca.crt"
      }
    },
    {
      "name": "test-cluster-1",
      "cluster": {
        "server": "KUBE_ADDRESS",
        "certificate-authority": "/etc/kubernetes/pki/ca.crt"
      }
    }
  ],
  "users": [
    {
      "name": "aws-user",
      "user": {
        "client-certificate": "/etc/kubernetes/pki/users/aws-user/aws-user.crt",
        "client-key": "/etc/kubernetes/pki/users/aws-user/aws-user.key"
      }
    },
    {
      "name": "dev-user",
      "user": {
        "client-certificate": "/etc/kubernetes/pki/users/dev-user/developer-user.crt",
        "client-key": "/etc/kubernetes/pki/users/dev-user/dev-user.key"
      }
    },
    {
      "name": "test-user",
      "user": {
        "client-certificate": "/etc/kubernetes/pki/users/test-user/test-user.crt",
        "client-key": "/etc/kubernetes/pki/users/test-user/test-user.key"
      }
    }
  ],
  "contexts": [
    {
      "name": "aws-user@kubernetes-on-aws",
      "context": {
        "cluster": "kubernetes-on-aws",
        "user": "aws-user"
      }
    },
    {
      "name": "research",
      "context": {
        "cluster": "test-cluster-1",
        "user": "dev-user"
      }
    },
    {
      "name": "test-user@development",
      "context": {
        "cluster": "development",
        "user": "test-user"
      }
    },
    {
      "name": "test-user@production",
      "context": {
        "cluster": "production",
        "user": "test-user"
      }
    }
  ],
  "current-context": "test-user@development"
}
```
**Expected Output:**

```json
[
  "aws-user",
  "dev-user",
  "test-user"
]
```

**JSON Path query:**
```bash
$.users.[:].name
```