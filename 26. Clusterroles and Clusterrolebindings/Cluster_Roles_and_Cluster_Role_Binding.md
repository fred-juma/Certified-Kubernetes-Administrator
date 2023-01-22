Cluster Roles and Cluster Role Binding

- Nodes are cluster wide reousrces and are not put in any particular namespace
- Resources are categorized as either namespaced or cluster scoped
- Namespaced resources include: pods, replicasets, jobs, deployments, services, secrets, roles, rolebindings, configmaps, PVC
- They are created in the default namespace unless another namespace is specified
- The right namespace has to be identified to create, view, describe or delete them
- Users are authorized to namespaced resources by means of roles and rolebindings
- To view a full list of namspaced resources:

```bash

controlplane ~ ➜  kubectl api-resources --namespaced=true
NAME                        SHORTNAMES   APIVERSION                     NAMESPACED   KIND
bindings                                 v1                             true         Binding
configmaps                  cm           v1                             true         ConfigMap
endpoints                   ep           v1                             true         Endpoints
events                      ev           v1                             true         Event
limitranges                 limits       v1                             true         LimitRange
persistentvolumeclaims      pvc          v1                             true         PersistentVolumeClaim
pods                        po           v1                             true         Pod
podtemplates                             v1                             true         PodTemplate
replicationcontrollers      rc           v1                             true         ReplicationController
resourcequotas              quota        v1                             true         ResourceQuota
secrets                                  v1                             true         Secret
serviceaccounts             sa           v1                             true         ServiceAccount
services                    svc          v1                             true         Service
controllerrevisions                      apps/v1                        true         ControllerRevision
daemonsets                  ds           apps/v1                        true         DaemonSet
deployments                 deploy       apps/v1                        true         Deployment
replicasets                 rs           apps/v1                        true         ReplicaSet
statefulsets                sts          apps/v1                        true         StatefulSet
localsubjectaccessreviews                authorization.k8s.io/v1        true         LocalSubjectAccessReview
horizontalpodautoscalers    hpa          autoscaling/v2                 true         HorizontalPodAutoscaler
cronjobs                    cj           batch/v1                       true         CronJob
jobs                                     batch/v1                       true         Job
leases                                   coordination.k8s.io/v1         true         Lease
endpointslices                           discovery.k8s.io/v1            true         EndpointSlice
events                      ev           events.k8s.io/v1               true         Event
helmchartconfigs                         helm.cattle.io/v1              true         HelmChartConfig
helmcharts                               helm.cattle.io/v1              true         HelmChart
addons                                   k3s.cattle.io/v1               true         Addon
pods                                     metrics.k8s.io/v1beta1         true         PodMetrics
ingresses                   ing          networking.k8s.io/v1           true         Ingress
networkpolicies             netpol       networking.k8s.io/v1           true         NetworkPolicy
poddisruptionbudgets        pdb          policy/v1                      true         PodDisruptionBudget
rolebindings                             rbac.authorization.k8s.io/v1   true         RoleBinding
roles                                    rbac.authorization.k8s.io/v1   true         Role
csistoragecapacities                     storage.k8s.io/v1              true         CSIStorageCapacity
ingressroutes                            traefik.containo.us/v1alpha1   true         IngressRoute
ingressroutetcps                         traefik.containo.us/v1alpha1   true         IngressRouteTCP
ingressrouteudps                         traefik.containo.us/v1alpha1   true         IngressRouteUDP
middlewares                              traefik.containo.us/v1alpha1   true         Middleware
middlewaretcps                           traefik.containo.us/v1alpha1   true         MiddlewareTCP
serverstransports                        traefik.containo.us/v1alpha1   true         ServersTransport
tlsoptions                               traefik.containo.us/v1alpha1   true         TLSOption
tlsstores                                traefik.containo.us/v1alpha1   true         TLSStore
traefikservices                          traefik.containo.us/v1alpha1   true         TraefikService

controlplane ~ ➜ 
```

- Cluster scoped objects include: nodes, PV, clusterroles, clusterrolebindings,certificatesigningrequests and namespaces
- To view a full list of clustered resources:
- Users are authorized to cluster wide resources by means of clusterroles and clusterrolebindings

```bash
controlplane ~ ➜  kubectl api-resources --namespaced=false
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
componentstatuses                 cs           v1                                     false        ComponentStatus
namespaces                        ns           v1                                     false        Namespace
nodes                             no           v1                                     false        Node
persistentvolumes                 pv           v1                                     false        PersistentVolume
mutatingwebhookconfigurations                  admissionregistration.k8s.io/v1        false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io/v1        false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io/v1                false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io/v1              false        APIService
tokenreviews                                   authentication.k8s.io/v1               false        TokenReview
selfsubjectaccessreviews                       authorization.k8s.io/v1                false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io/v1                false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io/v1                false        SubjectAccessReview
certificatesigningrequests        csr          certificates.k8s.io/v1                 false        CertificateSigningRequest
flowschemas                                    flowcontrol.apiserver.k8s.io/v1beta3   false        FlowSchema
prioritylevelconfigurations                    flowcontrol.apiserver.k8s.io/v1beta3   false        PriorityLevelConfiguration
nodes                                          metrics.k8s.io/v1beta1                 false        NodeMetrics
ingressclasses                                 networking.k8s.io/v1                   false        IngressClass
runtimeclasses                                 node.k8s.io/v1                         false        RuntimeClass
clusterrolebindings                            rbac.authorization.k8s.io/v1           false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io/v1           false        ClusterRole
priorityclasses                   pc           scheduling.k8s.io/v1                   false        PriorityClass
csidrivers                                     storage.k8s.io/v1                      false        CSIDriver
csinodes                                       storage.k8s.io/v1                      false        CSINode
storageclasses                    sc           storage.k8s.io/v1                      false        StorageClass
volumeattachments                              storage.k8s.io/v1                      false        VolumeAttachment

controlplane ~ ➜  
```


Create clusterroles and clusterrolebindings and specify rules


```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list", "get", "create", "delete"]


Then apply it with kubectl apply

rolebinding.yaml

After than link the clusterrole to a user by clusterrolebinding


```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: cluster-admin
```

Then apply it with kubectl apply

Note:
- Cluster role and cluster role binding can be created for namespaced resources as well. In that case the user will have access to these resource(s) across the namespaces

How many ClusterRoles do you see defined in the cluster?

```bash
controlplane ~ ➜  kubectl get ClusterRoles --no-headers | wc -l
69

controlplane ~ ➜  
```

How many ClusterRoleBindings exist on the cluster?

```bash
controlplane ~ ➜  kubectl get ClusterRoleBindings --no-headers | wc -l
54

controlplane ~ ➜  
```

What namespace is the cluster-admin clusterrole part of? **Cluster Roles are cluster wide and not part of any namespace**


What user/groups are the cluster-admin role bound to? The ClusterRoleBinding for the role is with the same name. **Group  system:masters**

```bash
controlplane ~ ➜  kubectl describe clusterrolebinding cluster-admin
Name:         cluster-admin
Labels:       kubernetes.io/bootstrapping=rbac-defaults
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
Role:
  Kind:  ClusterRole
  Name:  cluster-admin
Subjects:
  Kind   Name            Namespace
  ----   ----            ---------
  Group  system:masters  

controlplane ~ ➜  
```


What level of permission does the cluster-admin role grant? Inspect the cluster-admin role's privileges - **It perform any action on any resource in the cluster**

```bash
controlplane ~ ➜  kubectl describe clusterrole cluster-admin
Name:         cluster-admin
Labels:       kubernetes.io/bootstrapping=rbac-defaults
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  *.*        []                 []              [*]
             [*]                []              [*]

controlplane ~ ➜  
```

A new user michelle joined the team. She will be focusing on the nodes in the cluster. Create the required ClusterRoles and ClusterRoleBindings so she gets access to the nodes.

Create the clusterrole

```bash
controlplane ~ ➜  kubectl create clusterrole node-admin --verb=get,watch,list,create,delete --resource=nodes -o yaml --dry-run=client > cluster-admin.yaml
```

View the resulting yaml 

```bash
controlplane ~ ➜  vi cluster-admin.yaml 


apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: null
  name: node-admin
rules:
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - get
  - watch
  - list
  - create
  - delete
```

Apply the clusterrole definition

```bash
controlplane ~ ➜  kubectl apply -f cluster-admin.yaml 
Warning: resource clusterroles/cluster-admin is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
clusterrole.rbac.authorization.k8s.io/cluster-admin configured

```

Create the ClusterRoleBindings

```bash
controlplane ~ ➜  kubectl create clusterrolebinding node-admin-role-binding --clusterrole=node-admin --user=michelle -o yaml --dry-run=client > cluster-admin-role-binding.yaml
```

view the resulting definition yaml and update it to include the 

```bash
controlplane ~ ➜  vi cluster-admin-role-binding.yaml 

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  creationTimestamp: null
  name: node-admin-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: node-admin--user=michelle
```

Apply the definition

```bash
controlplane ~ ➜  kubectl apply -f cluster-role-binding.yaml 
clusterrolebinding.rbac.authorization.k8s.io/cluster-admin-role-binding created

controlplane ~ ➜  
```

Verify that michelle can view the nodes

```bash
controlplane ~ ➜  kubectl auth can-i list nodes --as michelle
Warning: resource 'nodes' is not namespace scoped

yes

controlplane ~ ➜  
```


michelle's responsibilities are growing and now she will be responsible for storage as well. Create the required ClusterRoles and ClusterRoleBindings to allow her access to Storage.

Get the API groups and resource names from command kubectl api-resources. Use the given spec:

-ClusterRole: storage-admin
- Resource: persistentvolumes
- Resource: storageclasses
- ClusterRoleBinding: michelle-storage-admin
- ClusterRoleBinding Subject: michelle
- ClusterRoleBinding Role: storage-admin

Get the API groups and resource names

```bash
controlplane ~ ➜  kubectl api-resources | grep storage
csidrivers                                     storage.k8s.io/v1                      false        CSIDriver
csinodes                                       storage.k8s.io/v1                      false        CSINode
csistoragecapacities                           storage.k8s.io/v1                      true         CSIStorageCapacity
storageclasses                    sc           storage.k8s.io/v1                      false        StorageClass
volumeattachments                              storage.k8s.io/v1                      false        VolumeAttachment

controlplane ~ ➜  
```

create the manifest yaml storage-admin.yaml

```bash
controlplane ~ ➜  vi storage-admin.yaml

controlplane ~ ➜  
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: storage-admin
rules:
- apiGroups:
  - ""
  resources:
  - persistentvolumes
  verbs:
  - get
  - watch
  - list
  - create
  - delete
- apiGroups:
  - "storage.k8s.io"
  resources:
  - storageclasses
  verbs:
  - get
  - watch
  - list
  - create
  - delete
```

```bash
controlplane ~ ➜  kubectl apply -f storage-admin.yaml 
clusterrole.rbac.authorization.k8s.io/storage-admin created

controlplane ~ ➜  
```

create the manifest yaml michelle-storage-admin.yaml

```bash
controlplane ~ ➜  vi michelle-storage-admin

controlplane ~ ➜  
```


```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: michelle-storage-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: storage-admin--user=michelle
```

```bash
controlplane ~ ➜  kubectl apply -f michelle-storage-admin 
clusterrolebinding.rbac.authorization.k8s.io/michelle-storage-admin created

controlplane ~ ➜ 
``` 


