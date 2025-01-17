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

### Ingress
- Ingress - data **entering** the system, i.e. visitor's request to view a webpage
- Egress - data **leaving** the system, i.e. file being downloaded from a server
Traffic is handled by proper services

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

