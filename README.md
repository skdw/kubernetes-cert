# kubernetes-cert
Kubernetes Certified Admin course notes

## LFS258

[Class forum](https://forum.linuxfoundation.org/categories/lfs258-class-forum)

By the end of this course, you will learn the following:

### The history and evolution of Kubernetes
- Kubernetes from Greek - pilot
- deployment of resources, cleaning them when no longer needed
- **decoupling** - one action does not require another to complete
- **transient** - system expects that all resources terminate, clean, replace
- objects go away and reconnect to their replacement -> flexible scalable environment
- K <eight letters> S -> *k8s*, Kate's
- Kubernetes implemented in Go language

### Its high-level architecture and components
- Instead of using a large server, Kubernetes approaches the same issue by deploying a large number of small servers, or microservices.
- Instead of a large Apache web server with many httpd daemons responding to page requests, there would be many nginx servers, each responding.
- A service ties traffic from one agent to another (for example, a frontend web server to a backend database)
- Containers provide a great way to package, ship, and run applications - that is the Docker motto.
- Docker - ease of building container images, simplicity of sharing images via registries, UI to manage containers
- Managing containers at scale and designing a distributed application based on microservices' principles may be challenging.
- They need to be written, or re-written, to be truly transient.

A good question to ponder: If you were to deploy Chaos Monkey, which could terminate any containers at any time, **would your customers notice**?

### The API, the most important resources that make the API, and how to use them
- Communication is entirely API call-driven, which allows for flexibility.
- Cluster configuration information is stored in a JSON format inside of `etcd`, but is most often written in YAML by the community.
- Kubernetes agents convert the YAML to JSON prior to persistence to the database.
- How to deploy and manage an application...
- Some upcoming features that will boost your productivity

### Other solutions - competitive to K8s
Docker Swarm, Apache Mesos, Nomad ...

### Borg heritage
[Google Cloud podcast](https://www.gcppodcast.com/post/episode-46-borg-and-k8s-with-john-wilkes/)

### Terminology
- **Containers** are part of `Pod` (larger object)
  - they share an IP address, access to storage, namespace
  - one container in a Pod runs app and others support the primary app
- Orchestration is managed through a series of watch-loops, called controllers or `operators` - mechanism to watch and react to changes in resources within a Kubernetes cluster
- Each controller interrogates the `kube-apiserver` for a particular object state
  - modifying the object until the declated state matches the current state
  - these controllers are compiled into the `kube-controller-manager`
- Default and feature-filled operator for containers is a `Deployment`
  - A Deployment does not directly work with pods. Instead it manages `ReplicaSets`.
  - Deployment can do rolling updates and rollbacks -> update without downtown / quickly revert previous version
- `ReplicaSet` is an operator which will create or terminate pods according to a `podSpec`.
  - It ensures that a specified number of identical copies (replicas) of your application are always running.
- `podSpec` is sent to the `kubelet`, which then interacts with the container engine
  - download and make available the required resources
  - spawn or terminate containers until the status matches the spec
- A service is used to communicate between pods, namespaces, and outside the cluster
- There are also Jobs and CronJobs to handle single or recurring tasks, among other default operators.

- `labels` - arbitrary strings which become part of the object metadata
- Nodes can have `taints` to discourage Pod assignments, unless the Pod has a `toleration` in its metadata

### Kubernetes architecture

Node types:
- **Control Plane** (cp), master nodes
  - `kube-apiserver` agent - all other agents send their requests to `kube-api` to authenticate, authorize and send them do destination
  - `kube-scheduler` - determines which node will host a Pod of containers
  - `etcd` database
  - state storage...
- **Worker** nodes (minions)
  - `kubelet` - primary agent: systemd process, receives requests to run the containers, download images, configurations, sends back status to `kube-apiserver`
  - `kube-proxy` - creates manages net rules to expose the container on the network

`etcd` is a consistent and highly-available key value store used as Kubernetes' backing store for all cluster data
only `kube-apiserver` talks to `etcd` database so that it is responsible for the persistent state of the database

[Fluentd](fluentd.org) - logging mechanism across cluster

#### Operators

Controllers / watch-loops

Simplified view is an agent, or Informer, and a downstream store
Using a `DeltaFIFO` queue, the source and downstream are compared
A loop process receives an `obj` / object, which is an array of deltas from the FIFO queue
As long as the delta is not of type `Deleted`, the logic of the operator is used to create or modify some object until it matches the specification

`DeltaFIFO` is like FIFO, but [allows you to process deletes](https://lairdnelson.wordpress.com/2018/01/07/understanding-kubernetes-tools-cache-package-part-4/)

- `Informer` - uses the API server as a source requests the state of an object via an API call
- `SharedInformer` - objects are often used by multiple other objects, creates a shared cache of the state for multiple requests
- `Workqueue` - uses a key to hand out tasks to various workers
- `endpoints`, `namespace`, `serviceaccounts` operators each manage the eponymous resources for Pods

#### Service operators

Flexible and scalable agents which **connect resources together** and will reconnect, should something die and a replacement is spawned. A service is an operator which listens to the endpoint operator to provide a persistent IP for Pods. Pods have ephemeral IP addresses chosen from a pool.

Service responsibilities:
- Connect Pods together
- Expose Pods to Internet
- Decouple settings
- Define Pod access policy

#### Pods

- Goal: orchestrate the lifecycle of a container, do not interact with particular containers
- **Pod** - smallest unit we can work with
- Design of a Pod typically follows a one-process-per-container architecture
- Containers in a Pod are started in parallel
- Only one IP address per Pod
- Communication within Pod: IPC / loopback / shared filesystem
- Sidecar - a container dedicated to performing a helper, i.e. logger

Pod template includes:
- apiVersion
- kind
- metadata
- spec

#### Rewrite legacy application

- Legacy - well known, high replacement cost
- Cloud-native - flexible, low outage effect, little traffic issues

#### Containers

While orchestration does not allow direct manipulation on a container level, we can manage the resources containers are allowed to consume

`PodSpec` / `resources` -> limits CPU, memory

#### Init Containers

Standard containers are sent to the container engine at the same time, and may start in any order
**Init container** must complete before app containers will be started

Init container can have a different view of the storage and security settings, which allows utilities and commands to be used, which the application would not be allowed to use.

Example:
- Run init container `sh -c until ls /db/dir; do sleep 5; done;`
- Run main app container that reads from database

#### Nodes

- One IP per Pod
- `NodePort` service connects the Pod to outside network
- Containers share the same namespace and the same IP address

#### Services

A Service connects one pod to another, or to outside of the cluster

Example: primary container App, optional sidecar Logger

#### Networking Setup

[Cluster Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)

The three main networking challenges to solve in a container orchestration system are:
- Coupled container-to-container communication (solved by the pod concept)
- Pod-to-pod communication -> software like Flannel
- External-to-pod communication

#### CNI Network Configuration File

To provide container networking, Kubernetes is standardizing on the Container Network Interface (CNI) specification.

[CNI GitHub](https://github.com/containernetworking/cni)

### APIs

REST-based API -> `kube-apiserver` communicates

`curl` query to the agent will expose the current API groups

json serialization
```
$ curl --cert userbob.pem --key userBob-key.pem \
--cacert /path/to/ca.pem \
https://k8sServer:6443/api/v1/pods
```
OpenAPI specification, Swagger UI - documentation of API, dynamic interaction

#### Check access
```
kubectl auth can-i create deployments --as bob
```
#### Annotations

Annotate pods - allow for metadata to be included with an object that may be helpful outside of the Kubernetes object interaction

key-value maps

`kubectl annotate pods --all description='Production Pods' -n prod`

Labels vs Annotations:
- Labels can be used to select objects and to find collections of objects that satisfy certain conditions
- Annotations are not used to identify and select objects

#### Verbose watch REST API requests

`kubectl --v=10 get pods firstpod`

### API Objects

- **Pod** (unit) - runs application container(s) with access to an IP address and storage
- **Service** (abstraction) - allows for IP traffic between pods, or outside world

Container < Pod < ReplicaSet < Deployment

#### Controllers

Controller - a control loop that watches the state of your cluster, then makes or requests changes where needed

- **Deployment** - controller that updates ReplicaSets, ensures their creation
- **ReplicaSet** - deploys pods of particular version or roll backs
- **DaemonSet** - ensures that a particular type of pod is deployed on each of nodes
  - useful to setup a logging daemon on each of nodes
  - always runs a single Pod on each node
- **StatefulSet** - each pod checked as an unique unit with ensured state; stable storage, stable network
- **HorizontalPodAutoscaler** - stable resource, scales Replication Controllers based on target CPU usage
- **VerticalPodAutoscaler** - adjusts the amount of CPU/memory requested by Pods
- **ClusterAutoscaler** - adds nodes to the cluster if unable to deploy a Pod; removes on low utilization
- **Job** - runs until the number of completions reached
- **CronJob** - runs on regular basis
- **Role Based Access Control (RBAC)** [[Security](#security)] - allows to define Roles withing a cluster and associate users to Roles, i.e. define a Role for someone who can only read pods in a specific namespace
  - ClusterRole
  - ClusterRoleBinding
  - RoleBinding

- **NetworkPolicy**
- **Ingress**
- **ConfigMap**
- **Node Controller**
- **PersistentVolumeClaim (PVC)**
- **EndpointSlice**

### Managing State with Deployments

- Most updates can be configured by editing a YAML file and running `kubectl apply`.
- In-use config modification with `kubectl edit`

When we create a deployment, then automatically:
- it creates a ReplicaSet
- ReplicaSet creates a Pod
- the Pod will download or run whatever container we have configured

We can then use a deployment to upgrade or roll back our ReplicaSet, steps:
- edit YAML file
- run `kubectl apply` or `kubectl edit`
- previous versions of the ReplicaSets are kept, allowing a rollback to return to a previous config

#### Labels

Arbitrary user-defined key-value pairs which can be attached to any resource, essential to administration.

#### Example deployment generation
```
$ kubectl create deployment dev-web --image=nginx:1.21
deployment "dev-web" created

To generate the YAML file of newly created objects, run the following command:
$ kubectl get deployments,rs,pods -o yaml

  apiVersion: v1
  items: ... # apiVersion kind (Deployment)
  metadata: ... # name labels annotations podAffinity
  spec: ... # Deployment spec: replicas matchLabels strategy
  template: # Pod template
    metadata: ...
    spec: ... # Pod containers restartPolicy
  status: ... # generated output: availableReplicas, conditions
```

#### Scaling and Rolling Updates
```
$ kubectl scale deploy/dev-web --replicas=4
deployment "dev-web" scaled

$ kubectl get deployments
NAME     READY   UP-TO-DATE  AVAILABLE  AGE
dev-web  4/4     4           4          20s

$ kubectl edit deployment dev-web
> opens text editor, triggers a rolling update
```

#### Rollbacks
```
$ kubectl create deploy ghost --image=ghost

$ kubectl annotate deployment/ghost kubernetes.io/change-cause="kubectl create deploy ghost --image=ghost"

$ kubectl get deployments ghost -o yaml
kubernetes.io/change-cause: kubectl..

$ kubectl set image deployment/ghost ghost=ghost:09 --all

$ kubectl get pods

NAME                    READY  STATUS            RESTARTS  AGE
ghost-2141819201-tcths  0/1    ImagePullBackOff  0         1m

$ kubectl rollout undo deployment/ghost ; kubectl get pods

NAME                    READY  STATUS   RESTARTS  AGE
ghost-3378155678-eq5i6  1/1    Running  0         7s

kubectl rolling-update # directly on ReplicationControllers, but this is done on the client side
# less safe - hence, if you close your client, the rolling update will stop
```

#### Using DaemonSets

This controller ensures that a single pod exists **on each node in the cluster**. Every Pod uses the same image.

Use `kind: DaemonSet`.

#### Labels
```
$ kubectl get pods -l run=ghost
NAME                    READY  STATUS   RESTARTS  AGE
ghost-3378155678-eq5i6  1/1    Running  0         10m   # assigned a label
```

### Helm and Kustomize

- Helm - a package manager for Kubernetes (like yum, apt)
  - A standard way to manage Kubernetes applications, reducing the complexity of deploying & maintaining
  - Chart - similar to a package
  - Tarball put in a repository for deployment and sharing
  - Template - installation and removal scripts - **manifests**
  - Deployment created based on the Chart
  - Charts can be customized using `values.yaml` files or cmd arguments
  - Integrates well with CI/CD pipelines
  - Chart repositories, [Artifact Hub](https://artifacthub.io/)
  - Deploying a Chart: `helm install`
- Kustomize
  - Overcomes the limitations of using pure templating systems
  - You can define a base set of resources and then apply overlays
  - A more modular and maintainable approach to managing k8s configurations
  - Strategic merging and patches instead of templating
  - Is a built-in feature: `kubectl kustomize dir`, `kubectl apply -k dir`
  - `kustomization.yaml` lists the resources to manage and instructions on how to modify them
  - For instance: base contains the common deployment config, while overlays can adjust replicaCounts or image tags
  - Patches let you apply fine-grained changes
  - Includes `configMapGenerator`, `secretGenerator` - creating resources dynamically

### Volumes and Data

- The persistent storage available to Kubernetes
- Many backend options available: local storage, Ceph, dynamic storage provider AWS Google
- When storage is given to a Pod, all of its containers inside have equal access
- Containers can talk to each other by writing to / reading from shared storage

#### Volume

- A Kubernetes volume shares the Pod lifetime, not the containers within. Should a container terminate, the data would **continue to be available** to the new container.
- Container Storage Interface (CSI) is a standard API that enables k8s to connect with various storage systems.
- Each Volume requires a name, a type and a mount point.
- Access modes: `ReadWriteOnce` (rw by a single node), `ReadOnlyMany` (ro by multiple nodes), `ReadWriteMany` (rw by multiple nodes)
- Volume types:
  - Cloud CGE `GCEpersistentDisk`, AWS `awsElasticBlockStore` - mount disks in Pods
  - `emptydir` - empty directory that gets erased when the Pod dies
  - `hostPath` - mounts a resource from the host node filesystem: directory, file socket, block device
  - `DirectoryOrCreate` `FileOrCreate` create the resources on the host
  - `NFS` `iSCSI` network interfaces
  - `azureDisk` `azureFile` `csi` `gitRepo` `local` `persistentVolumeClaim` etc - CSI allows for even more flexibility

#### Persistent Volume

- A storage abstraction used to retain data longer than the Pod using it.
- Allows for empty or pre-populated volumes to be claimed by a Pod using `PersistentVolumeClaim`, then outlive the Pod.
- Data inside can be used by another Pod, or as a means of retrieving data.

- `kubectl get pv` - actual physical or virtual storage resource, analogy: parking spot
- `kubectl get pvc` - request for storage by a user (or a Pod), analogy: parking ticket

`ResourceQuota` can limit PVC count and usage

`StorageClass` `kubectl get sc` - enables dynamic provisioning of storage i.e. NFS client

Upon release - Persistent Volume Reclaim Policy: Retain / Recycle / Delete (default);  whether to keep/remove pv on pvc removed

#### Secrets / ConfigMaps

- These are two API objects to provide data to a Pod
- Secret data is **base64 encoded**; it is not encrypted! It can be used as an env variable in a Pod
- ConfigMap data is **neither encrypted**, nor encoded; just plain raw
```
kubectl create secret generic mysql --from-literal=password=root
```
For encryption, `EncryptionConfiguration` is needed with a key and proper identity
Then, the kube-apiserver needs the `--encryption-provider-config` flag set to a provider such as kms

### Services

A service is an agent for exposing Pods - it gets traffic from the outside world to a pod, or from one pod to another.

Four types of services:
- `ClusterIP` - internally-facing IP address
- `NodePort` - static IP address accessible to the outside world
- `LoadBalancer` - similar to `NodePort` but spreads the traffic to multiple Pods; handles pods replacements
- `ExternalName` - an alias for interacting with DNS
```
$ kubectl expose deployment/nginx --port=80 --type=NodePort

$ kubectl get svc

NAME        TYPE       CLUSTER-IP  EXTERNAL-IP  PORT(S)        AGE
kubernetes  ClusterIP  10.0.0.1    <none>       443/TCP        18h
nginx       NodePort   10.0.0.112  <none>       80:31230/TCP   5s

$ kubectl get svc nginx -o yaml
```
The service used port 80 and generated a random port on all the nodes -> `get svc` shows a random port 31230 exposed

`kubectl proxy` -> `Starting to serve at 127.0.0.1:8001` -> a local proxy for development
  - a quick way to check the services
  - captures the shell, unless in the background

DNS registration: tools `nslookup`, `dig`, `nc`, `wireshark`...

### Ingress

A controller allowing to expose Pods similarly to services, but more efficiently. Instead of individual services for each of the pods, we can set up a controller that handles all of the traffic.

When a new endpoint is created, the daemon uses the set of rules to allow inbound connection to a service.

- Ingress - data **entering** the system, i.e. visitor's request to view a webpage
  - Internet -> **Ingress** -> Services
- Egress - data **leaving** the system, i.e. file being downloaded from a server
Traffic is handled by proper services

Ingress is an API object: `kubectl get ingress`

[Istio](https://istio.io/) service mesh: load balancing, service-to-service authentication, monitoring

Ingress limitations: Support only HTTP(S) routing and TLS termination, no native support for TCP, gRPC...

#### Gateway API

Imagine you have a big building (your Kubernetes cluster) and you want to control how people (network traffic) get in and where they go inside.

Ingress (the old way): Think of Ingress as a single, somewhat limited front door. It could get people in, but if you wanted fancy rules like "only people wearing red hats go to the kitchen," it was hard to set up.

Gateway API (the new, better way): This is like having a whole, smart reception area with different types of doors and a clear system for guiding people.

### Scheduling

Affect where pods will be deployed based on labels features

`kube-scheduler` agent schedules Pod placements: it looks at nodes' labels to determine which nodes will run a Pod -> **affinity** and **anti-affinity**

- **Taint** - label applied to **Nodes** that tells certain pods to avoid the node
- **Toleration** - similar, applied to **Pods**; if a node has been tainted, a Pod requires a toleration to be deployed
- **NodeSelector** - call out a particular node for a pod to run on
- We can also deploy our own custom scheduler, and run it alongside of the existing `kube-scheduler` agent

Node selection in `kube-scheduler`:
- *Filtering stage*: identifies the set of Nodes where the Pod can be scheduled
- *Scoring stage*: rates the remaining nodes to determine the best Pod placement
- The Pod is given to the Node with the highest ranking by `kube-scheduler`.

Scheduling profiles available, plugins can be used: `ImageLocality`, `TaintToleration`, `NodeName`, `NodePorts`, `NodeAffinity`...

API object `KubeSchedulerConfiguration`, with profiles scheduler name, plugins...

Pods can include scheduler name in its `.spec.schedulerName`

[EXAM] Scheduling for maintenance / cluster update
- `kubectl drain NODE` - drain node in preparation for maintenance
- `kubectl cordon NODE` - mark node as unschedulable

Command to view the scheduler: `kubectl get events`

### Logging & Troubleshooting

K8s does not have a built-in cluster-wide logging utility. Instead, secondary products to aggregate (both CNCF projects):
- **Fluentd** - logs aggregation & processing
- **Prometheus** - metric aggregation, Grafana visualizations

Metic Server: collects CPU usage, memory, disk usage -> exposes a standard API `/apis/metrics/k8s.io/`, which can be consumed by other agents, such as autoscalers

Basic flow of troubleshooting:
- Use standard linux tools. If not available, deploy a similar tool, like `busybox`.
  ```
  $ kubectl create deploy busybox --image=busybox -- sleep 3600
  $ kubectl exec -ti <busybox_pod> -- /bin/sh
  ```
- `kubectl logs pod-name`

#### Cluster start sequence

- `systemctl status kubelet.service`
  - `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`
- `staticPodPath` indicates the directory where kubelet will read every yaml file and start every pod

#### kubectl plugin manager - `krew`

- `kubectl krew search`
- `kubectl krew install tail`

Traffic monitoring with `sniff` plugin + Wireshark

### Custom resource definitions

Two ways to add custom resources:
- Adding a new object to the existing cluster
  - `kind: CustomResourceDefinition` -> `kind: BackUp` (a custom defined kind)
  - A CRD allows the resource to be deployed in a namespace or be available in the entire cluster.
- Aggregated APIs, add a secondary API server to the cluster
  - API calls accepted by `kube-apiserver` and forwarded to the secondary API server to be handled

### Security

LFS260 - a separate course on k8s security

How Pods use security policies, like SELinux

On each API request, `kube-apiserver` goes through a 3-step process:
- Authentication (X509 certificate, a token or a webhook to an LDAP server)
- Authorization (RBAC - role-based access control)
- Admission controller (8-9 shell scripts that can do many things)

Security contexts limit what processes running in containers can do. For example, the UID of the process, the Linux capabilities, and the filesystem group can be limited. e.g. `runAsNonRoot`

Network policies - by default all pods can reach each other; however isolation can be introduced, e.g. `NetworkPolicy -> ingress -> ipBlock`

### High Availability

kubeadm provides the integrated ability to join multiple cp nodes with collocated etcd databases -> higher redundancy, fault tolerance

To ensure that workers and other control planes continue to have access, it is a good idea to use a load balancer.

- Collocated databases - two or more cp servers joined to the cluster
- Non-collocated databases - using an external cluster of etcd allows for less interruption should a node fail (however setup takes more work to configure)

## Labs

`killer.sh` exam simulator - 2 attempts

### Getting started

#### Tools

- `minikube start` / `minikube dashboard` - run k8s locally
- `helm` - for using Kubernetes, to search for and install software using charts
- `kubeadm` - create a cluster
- `flannel` - a network fabric for containers, designed for Kubernetes
- `ansible` `kubespray` / `terraform` - provision resources, deploy clusters

#### Context
```
kubectl config get-contexts
kubectl config use-context arn:aws:eks:<EKS_CLUSTER>
kubectl config use-context minikube

~/.kube/config contains:
  - apiVersion -> kube-apiserver where to assign the data
  - clusters -> name of the cluster, where to send the API calls
  - contexts -> one config file to set namespace, user, cluster
  - current-context -> which cluster and user the kubectl command would use
  - kind -> object type `Config`
  - prferences -> optional, like colorizing output
  - users -> nickname associated with credentials
```
#### Connect to Google Cloud
- Create a VPC network
- Create a firewall rule - allow traffic from IP range
- Compute Engine -> Create VM Instance
- Choose network interface - created VPC
- Create instance / equivalent command line / equivalent REST
- Paste SSH key

### Installation & configuration
```
# Simple get, create, describe
kubectl get pods,deployments,services,events,logs
kubectl get deployment -o yaml > first.yaml
kubectl create deployment -f first.yaml
kubectl describe deployment

# Dump to yaml, re-deploy modifies
kubectl get deployment nginx -o yaml > second.yaml
diff first.yaml second.yaml
kubectl create deployment two --image=nginx --dry-run=client -o yaml

# Create a service to view the welcome page
kubectl expose deployment/nginx
> error: couldn't find port via --port flag or introspection
vim first.yaml
> ports:
  - containerPort: 80
    protocol: TCP
kubectl replace -f first.yaml --force

kubectl get deploy,pod
kubectl expose deployment/nginx
> service/nginx exposed
kubectl get svc nginx  # get service
kubectl get ep nginx   # get endpoint
curl ip_address:80
```
#### Access from outside the cluster
```
# Print env
kubectl exec nginx-1423793266-13p69 \
  -- printenv |grep KUBERNETES
...
kubectl expose deployment nginx --type=LoadBalancer
kubectl get svc
curl ip_address:80
```
#### Scale
```
kubectl scale deployment nginx --replicas=0 # terminate
kubectl scale deployment nginx --replicas=2 # restart
```
VM instances: master, worker

### Architecture labs

#### [EXAM] Backup the `etcd` database

`kubectl -n kube-system exec -it etcd-cp ...`

#### [EXAM] Perform cluster upgrade

- `kubectl drain cp` - prepare for maintenance
- `sudo kubeadm upgrade plan` - pre-flight checks
- `sudo kubeadm upgrade apply v1.31.1`

#### [EXAM] Working with CPU and Memory Constraints

- YAML output to configuration file
- vim `hog.yaml`
- `resources` `limits` `requests`
- `kubectl replace -f hog.yml`
- `kubectl get po`
- `kubectl logs <POD_NAME>` -> Allocating memory logs

#### [EXAM] Resource limits for a namespace

- `kubectl create namespace`
- `vim low-resource-range.yaml`
- `kubectl create -f low-resource-range.yaml -n low-usage-limit`
- `kubectl -n low-usage-limit create deployment...` - create in namespace

### API labs

#### TLS Access

curl/golang client can be run with TLS keys

calls to the `kube-apiserver` get or set a PodSpec, or desired state

grab keys from `~/.kube/config`:
 - `client-cert`
 - `client-key-data`
 - `certificate-authority-data`

encode the keys: `echo $client | base64 -d - > ./client.pem`

API server URL: `kubectl config view | grep server`

CURL with cert:
```sh
curl --cert ./client.pem \
--key ./client-key.pem --cacert ./ca.pem \
https://k8scp:6443/api/v1/pods```

CURL POST request `curlpod.json` - create a new pod:
```sh
curl --cert ./client.pem \
--key ./client-key.pem --cacert ./ca.pem \
https://k8scp:6443/api/v1/namespaces/default/pods \
-XPOST -H'Content-Type: application/json' \
-d@curlpod.json
```
#### Explore API calls

`strace` - system call tracer; monitor and tamper with instructions between processes and the Linux kernel
```
strace kubectl get endpoints
```
`.kube/cache/discovery` ... `python -m json.tool v1/serverresources.json | grep kind`

#### RESTful API Access

With Bearer token
```
kubectl config view -> IP and port
export token=$(kubectl create token default)

# Check API-s
curl https://k8scp:6443/apis --header "Authorization: Bearer $token" -k

# Check API v1
curl https://k8scp:6443/api/v1 --header "Authorization: Bearer $token" -k

# Get a list of namespaces -> permission error
curl \https://k8scp:6443/api/v1/namespaces --header "Authorization: Bearer $token" -k
```
#### Using the Proxy

Proxy can be run from a node or within a Pod through the use of a sidecar

Deploy a proxy listening to the loopback address, use `curl` to address the API server, no need to authenticate
```
kubectl proxy -h
kubectl proxy --api-prefix=/ &
-> [1] 22500
   Starting to serve on 127.0.0.1:8001

# Now proxy makes the requests on our behalf, access is granted
curl http://127.0.0.1:8001/api/
curl http://127.0.0.1:8001/api/v1/namespaces

kill 22500
```
#### Working with Jobs

Run objects a particular number of times
```
job.yaml
  kind: Job
  metadata:
    name: sleepy
  ...
      command: ["/bin/sleep"]
      args: ["3"]

kubectl create -f job.yaml
kubectl get job

kubectl get jobs.batch sleepy -o yaml
> spec parameters to affect how the job runs
  - completions -> number of target task completions
  - parallelism -> how many tasks at a time
  - backoffLimit -> how many failures allowed
  - activeDeadlineSeconds -> break task after a given time
```

## CKA Exam

### [CKA Curriculum](https://github.com/cncf/curriculum/blob/master/CKA_Curriculum_v1.33.pdf)

### [Exam resources](https://docs.linuxfoundation.org/tc-docs/certification/certification-resources-allowed#certified-kubernetes-administrator-cka-and-certified-kubernetes-application-developer-ckad) - available during the exam

### How I passed
- [YouTube](https://www.youtube.com/watch?v=dHXgg9fbP8E)
- [Blog](https://cloudchamp.notion.site/How-I-Passed-my-CKA-Exam-55fef633b454438aadc54a7261312ec9)

### [Exam questions](https://skillcertpro.com/product/certified-kubernetes-administrator-cka-exam-questions/)

### [Practice exercises GitHub repo](https://github.com/alijahnas/CKA-practice-exercises/)

### TOPICS to absolutely know before sitting the exam

1. [ETCD Backup & Restore](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)
2. [Cluster Upgrade](https://kubernetes.io/docs/tasks/administer-cluster/cluster-upgrade/)
3. [Working with PV and PVC](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)
4. [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
5. [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
6. [Jsonpath & Custom columns](https://kubernetes.io/docs/reference/kubectl/quick-reference/)
7. [Monitoring/Logging Pods and nodes](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_top/)
8. VIM & Nano
9. [Service Accounts, Role and Role Bindings](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)

