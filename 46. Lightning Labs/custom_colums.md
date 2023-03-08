#### Task

Print the names of all deployments in the admin2406 namespace in the following format:

DEPLOYMENT CONTAINER_IMAGE READY_REPLICAS NAMESPACE

<deployment name> <container image used> <ready replica count> <Namespace>
. The data should be sorted by the increasing order of the deployment name.

Example:

DEPLOYMENT CONTAINER_IMAGE READY_REPLICAS NAMESPACE
deploy0 nginx:alpine 1 admin2406
Write the result to the file /opt/admin2406_data.

#### Solution

List the deployments on the specified namespace

```bash

controlplane ~ ➜  kubectl get deployment -n admin2406 -o wide
NAME      READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
deploy1   1/1     1            1           20m   nginx        nginx          app=deploy1
deploy2   1/1     1            1           20m   nginx        nginx:alpine   app=deploy2
deploy3   1/1     1            1           20m   nginx        nginx:1.16     app=deploy3
deploy4   1/1     1            1           20m   nginx        nginx:1.17     app=deploy4
deploy5   1/1     1            1           20m   nginx        nginx:latest   app=deploy5

controlplane ~ ➜  
```

Print the details of deployments in the cluster  in json format

```json
controlplane ~ ➜  kubectl get deployment -n admin2406 -o wide -o json

{
    "apiVersion": "v1",
    "items": [
        {
            "apiVersion": "apps/v1",
            "kind": "Deployment",
            "metadata": {
                "annotations": {
                    "deployment.kubernetes.io/revision": "1"
                },
                "creationTimestamp": "2023-03-08T07:31:46Z",
                "generation": 1,
                "labels": {
                    "app": "deploy1"
                },
                "name": "deploy1",
                "namespace": "admin2406",
                "resourceVersion": "6957",
                "uid": "ca715382-8892-4cf6-9000-56b48329a606"
            },
            "spec": {
                "progressDeadlineSeconds": 600,
                "replicas": 1,
                "revisionHistoryLimit": 10,
                "selector": {
                    "matchLabels": {
                        "app": "deploy1"
                    }
                },
                "strategy": {
                    "rollingUpdate": {
                        "maxSurge": "25%",
                        "maxUnavailable": "25%"
                    },
                    "type": "RollingUpdate"
                },
                "template": {
                    "metadata": {
                        "creationTimestamp": null,
                        "labels": {
                            "app": "deploy1"
                        }
                    },
                    "spec": {
                        "containers": [
                            {
                                "image": "nginx",
                                "imagePullPolicy": "Always",
                                "name": "nginx",
                                "resources": {},
                                "terminationMessagePath": "/dev/termination-log",
                                "terminationMessagePolicy": "File"
                            }
                        ],
                        "dnsPolicy": "ClusterFirst",
                        "restartPolicy": "Always",
                        "schedulerName": "default-scheduler",
                        "securityContext": {},
                        "terminationGracePeriodSeconds": 30
                    }
                }
            },
            "status": {
                "availableReplicas": 1,
                "conditions": [
                    {
                        "lastTransitionTime": "2023-03-08T07:31:46Z",
                        "lastUpdateTime": "2023-03-08T07:31:48Z",
                        "message": "ReplicaSet \"deploy1-78d8d47657\" has successfully progressed.",
                        "reason": "NewReplicaSetAvailable",
                        "status": "True",
                        "type": "Progressing"
                    },
                    {
                        "lastTransitionTime": "2023-03-08T07:49:03Z",
                        "lastUpdateTime": "2023-03-08T07:49:03Z",
                        "message": "Deployment has minimum availability.",
                        "reason": "MinimumReplicasAvailable",
                        "status": "True",
                        "type": "Available"
                    }
                ],
                "observedGeneration": 1,
                "readyReplicas": 1,
                "replicas": 1,
                "updatedReplicas": 1
            }
        },
        {
            "apiVersion": "apps/v1",
            "kind": "Deployment",
            "metadata": {
                "annotations": {
                    "deployment.kubernetes.io/revision": "1"
                },
                "creationTimestamp": "2023-03-08T07:31:46Z",
                "generation": 1,
                "labels": {
                    "app": "deploy2"
                },
                "name": "deploy2",
                "namespace": "admin2406",
                "resourceVersion": "6949",
                "uid": "dd678a65-2723-475c-97bb-d5bb9c79feb8"
            },
            "spec": {
                "progressDeadlineSeconds": 600,
                "replicas": 1,
                "revisionHistoryLimit": 10,
                "selector": {
                    "matchLabels": {
                        "app": "deploy2"
                    }
                },
                "strategy": {
                    "rollingUpdate": {
                        "maxSurge": "25%",
                        "maxUnavailable": "25%"
                    },
                    "type": "RollingUpdate"
                },
                "template": {
                    "metadata": {
                        "creationTimestamp": null,
                        "labels": {
                            "app": "deploy2"
                        }
                    },
                    "spec": {
                        "containers": [
                            {
                                "image": "nginx:alpine",
                                "imagePullPolicy": "IfNotPresent",
                                "name": "nginx",
                                "resources": {},
                                "terminationMessagePath": "/dev/termination-log",
                                "terminationMessagePolicy": "File"
                            }
                        ],
                        "dnsPolicy": "ClusterFirst",
                        "restartPolicy": "Always",
                        "schedulerName": "default-scheduler",
                        "securityContext": {},
                        "terminationGracePeriodSeconds": 30
                    }
                }
            },
            "status": {
                "availableReplicas": 1,
                "conditions": [
                    {
                        "lastTransitionTime": "2023-03-08T07:31:46Z",
                        "lastUpdateTime": "2023-03-08T07:31:49Z",
                        "message": "ReplicaSet \"deploy2-d6b48767c\" has successfully progressed.",
                        "reason": "NewReplicaSetAvailable",
                        "status": "True",
                        "type": "Progressing"
                    },
                    {
                        "lastTransitionTime": "2023-03-08T07:49:03Z",
                        "lastUpdateTime": "2023-03-08T07:49:03Z",
                        "message": "Deployment has minimum availability.",
                        "reason": "MinimumReplicasAvailable",
                        "status": "True",
                        "type": "Available"
                    }
                ],
                "observedGeneration": 1,
                "readyReplicas": 1,
                "replicas": 1,
                "updatedReplicas": 1
            }
        },
        {
            "apiVersion": "apps/v1",
            "kind": "Deployment",
            "metadata": {
                "annotations": {
                    "deployment.kubernetes.io/revision": "1"
                },
                "creationTimestamp": "2023-03-08T07:31:46Z",
                "generation": 1,
                "labels": {
                    "app": "deploy3"
                },
                "name": "deploy3",
                "namespace": "admin2406",
                "resourceVersion": "6952",
                "uid": "3ba85c86-02eb-4ab9-92fe-0257fbcd7748"
            },
            "spec": {
                "progressDeadlineSeconds": 600,
                "replicas": 1,
                "revisionHistoryLimit": 10,
                "selector": {
                    "matchLabels": {
                        "app": "deploy3"
                    }
                },
                "strategy": {
                    "rollingUpdate": {
                        "maxSurge": "25%",
                        "maxUnavailable": "25%"
                    },
                    "type": "RollingUpdate"
                },
                "template": {
                    "metadata": {
                        "creationTimestamp": null,
                        "labels": {
                            "app": "deploy3"
                        }
                    },
                    "spec": {
                        "containers": [
                            {
                                "image": "nginx:1.16",
                                "imagePullPolicy": "IfNotPresent",
                                "name": "nginx",
                                "resources": {},
                                "terminationMessagePath": "/dev/termination-log",
                                "terminationMessagePolicy": "File"
                            }
                        ],
                        "dnsPolicy": "ClusterFirst",
                        "restartPolicy": "Always",
                        "schedulerName": "default-scheduler",
                        "securityContext": {},
                        "terminationGracePeriodSeconds": 30
                    }
                }
            },
            "status": {
                "availableReplicas": 1,
                "conditions": [
                    {
                        "lastTransitionTime": "2023-03-08T07:31:46Z",
                        "lastUpdateTime": "2023-03-08T07:31:53Z",
                        "message": "ReplicaSet \"deploy3-5bdcc4b88c\" has successfully progressed.",
                        "reason": "NewReplicaSetAvailable",
                        "status": "True",
                        "type": "Progressing"
                    },
                    {
                        "lastTransitionTime": "2023-03-08T07:49:03Z",
                        "lastUpdateTime": "2023-03-08T07:49:03Z",
                        "message": "Deployment has minimum availability.",
                        "reason": "MinimumReplicasAvailable",
                        "status": "True",
                        "type": "Available"
                    }
                ],
                "observedGeneration": 1,
                "readyReplicas": 1,
                "replicas": 1,
                "updatedReplicas": 1
            }
        },
        {
            "apiVersion": "apps/v1",
            "kind": "Deployment",
            "metadata": {
                "annotations": {
                    "deployment.kubernetes.io/revision": "1"
                },
                "creationTimestamp": "2023-03-08T07:31:47Z",
                "generation": 1,
                "labels": {
                    "app": "deploy4"
                },
                "name": "deploy4",
                "namespace": "admin2406",
                "resourceVersion": "6936",
                "uid": "b26ef3a2-8191-4fce-bf65-3a6ae121f140"
            },
            "spec": {
                "progressDeadlineSeconds": 600,
                "replicas": 1,
                "revisionHistoryLimit": 10,
                "selector": {
                    "matchLabels": {
                        "app": "deploy4"
                    }
                },
                "strategy": {
                    "rollingUpdate": {
                        "maxSurge": "25%",
                        "maxUnavailable": "25%"
                    },
                    "type": "RollingUpdate"
                },
                "template": {
                    "metadata": {
                        "creationTimestamp": null,
                        "labels": {
                            "app": "deploy4"
                        }
                    },
                    "spec": {
                        "containers": [
                            {
                                "image": "nginx:1.17",
                                "imagePullPolicy": "IfNotPresent",
                                "name": "nginx",
                                "resources": {},
                                "terminationMessagePath": "/dev/termination-log",
                                "terminationMessagePolicy": "File"
                            }
                        ],
                        "dnsPolicy": "ClusterFirst",
                        "restartPolicy": "Always",
                        "schedulerName": "default-scheduler",
                        "securityContext": {},
                        "terminationGracePeriodSeconds": 30
                    }
                }
            },
            "status": {
                "availableReplicas": 1,
                "conditions": [
                    {
                        "lastTransitionTime": "2023-03-08T07:31:47Z",
                        "lastUpdateTime": "2023-03-08T07:31:56Z",
                        "message": "ReplicaSet \"deploy4-6b96456775\" has successfully progressed.",
                        "reason": "NewReplicaSetAvailable",
                        "status": "True",
                        "type": "Progressing"
                    },
                    {
                        "lastTransitionTime": "2023-03-08T07:49:02Z",
                        "lastUpdateTime": "2023-03-08T07:49:02Z",
                        "message": "Deployment has minimum availability.",
                        "reason": "MinimumReplicasAvailable",
                        "status": "True",
                        "type": "Available"
                    }
                ],
                "observedGeneration": 1,
                "readyReplicas": 1,
                "replicas": 1,
                "updatedReplicas": 1
            }
        },
        {
            "apiVersion": "apps/v1",
            "kind": "Deployment",
            "metadata": {
                "annotations": {
                    "deployment.kubernetes.io/revision": "1"
                },
                "creationTimestamp": "2023-03-08T07:31:47Z",
                "generation": 1,
                "labels": {
                    "app": "deploy5"
                },
                "name": "deploy5",
                "namespace": "admin2406",
                "resourceVersion": "6954",
                "uid": "d87b1d13-9aad-4832-9e97-a12cf89f8e36"
            },
            "spec": {
                "progressDeadlineSeconds": 600,
                "replicas": 1,
                "revisionHistoryLimit": 10,
                "selector": {
                    "matchLabels": {
                        "app": "deploy5"
                    }
                },
                "strategy": {
                    "rollingUpdate": {
                        "maxSurge": "25%",
                        "maxUnavailable": "25%"
                    },
                    "type": "RollingUpdate"
                },
                "template": {
                    "metadata": {
                        "creationTimestamp": null,
                        "labels": {
                            "app": "deploy5"
                        }
                    },
                    "spec": {
                        "containers": [
                            {
                                "image": "nginx:latest",
                                "imagePullPolicy": "Always",
                                "name": "nginx",
                                "resources": {},
                                "terminationMessagePath": "/dev/termination-log",
                                "terminationMessagePolicy": "File"
                            }
                        ],
                        "dnsPolicy": "ClusterFirst",
                        "restartPolicy": "Always",
                        "schedulerName": "default-scheduler",
                        "securityContext": {},
                        "terminationGracePeriodSeconds": 30
                    }
                }
            },
            "status": {
                "availableReplicas": 1,
                "conditions": [
                    {
                        "lastTransitionTime": "2023-03-08T07:31:47Z",
                        "lastUpdateTime": "2023-03-08T07:31:57Z",
                        "message": "ReplicaSet \"deploy5-5dc4d5d78b\" has successfully progressed.",
                        "reason": "NewReplicaSetAvailable",
                        "status": "True",
                        "type": "Progressing"
                    },
                    {
                        "lastTransitionTime": "2023-03-08T07:49:03Z",
                        "lastUpdateTime": "2023-03-08T07:49:03Z",
                        "message": "Deployment has minimum availability.",
                        "reason": "MinimumReplicasAvailable",
                        "status": "True",
                        "type": "Available"
                    }
                ],
                "observedGeneration": 1,
                "readyReplicas": 1,
                "replicas": 1,
                "updatedReplicas": 1
            }
        }
    ],
    "kind": "List",
    "metadata": {
        "resourceVersion": ""
    }
}
```

Construct the custom columns in the specified order

```bash

controlplane ~ ➜  kubectl get deployment -n admin2406 -o=custom-columns=DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[:].image,READY_REPLICAS:.status.readyReplicas,NAMESPACE:.metadata.namespace
DEPLOYMENT   CONTAINER_IMAGE   READY_REPLICAS   NAMESPACE
deploy1      nginx             1                admin2406
deploy2      nginx:alpine      1                admin2406
deploy3      nginx:1.16        1                admin2406
deploy4      nginx:1.17        1                admin2406
deploy5      nginx:latest      1                admin2406
```


Print the output in the specified file

```bash
controlplane ~ ➜  kubectl get deployment -n admin2406 -o=custom-columns=DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[:].image,READY_REPLICAS:.status.readyReplicas,NAMESPACE:.metadata.namespace > /opt/admin2406_data

controlplane ~ ➜  
```

Confirm the printed file contents are as per instructions

```bash
controlplane ~ ➜  cat /opt/admin2406_data
DEPLOYMENT   CONTAINER_IMAGE   READY_REPLICAS   NAMESPACE
deploy1      nginx             1                admin2406
deploy2      nginx:alpine      1                admin2406
deploy3      nginx:1.16        1                admin2406
deploy4      nginx:1.17        1                admin2406
deploy5      nginx:latest      1                admin2406

controlplane ~ ➜ 
```

***The end*** 

