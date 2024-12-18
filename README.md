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

### Kubernetes architecture
Every node running a container would have two processes:
- `kubelet` - systemd process, receives requests to run the containers
- `kube-proxy` - creates manages net rules to expose the container on the network

Node types:
- control plane (cp) nodes - run API server, scheduler, state storage...
- worker nodes (minions)

`etcd` is a consistent and highly-available key value store used as Kubernetes' backing store for all cluster data

### Terminology
- **Containers** are part of `Pod` (larger object)
  - they share an IP address, access to storage, namespace
  - one container in a Pod runs app and others support the primary app
- Orchestration is managed through a series of watch-loops, called controllers or `operators`
- Each controller interrogates the `kube-apiserver` for a particular object state
  - modifying the object until the declated state matches the current state
  - these controllers are compiled into the `kube-controller-manager`
- Default and feature-filled operator for containers is a `Deployment`
  - A Deployment does not directly work with pods. Instead it manages `ReplicaSets`
- `ReplicaSet` is an operator which will create or terminate pods according to a `podSpec`
- `podSpec` is sent to the `kubelet`, which then interacts with the container engine 
  - download and make available the required resources
  - spawn or terminate containers until the status matches the spec
- A service is used to communicate between pods, namespaces, and outside the cluster
- There are also Jobs and CronJobs to handle single or recurring tasks, among other default operators.

- `labels` - arbitrary strings which become part of the object metadata
- Nodes can have `taints` to discourage Pod assignments, unless the Pod has a `toleration` in its metadata

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
```
#### Connect to Google Cloud
- Create a VPC network
- Create a firewall rule - allow traffic from IP range
- Compute Engine -> Create VM Instance
- Choose network interface - created VPC
- Create instance / equivalent command line / equivalent REST
- Paste SSH key

### Ingress
- Ingress - data **entering** the system, i.e. visitor's request to view a webpage
- Egress - data **leaving** the system, i.e. file being downloaded from a server

VM instances: master, worker

## CKA Exam

### How I passed
- [YouTube](https://www.youtube.com/watch?v=dHXgg9fbP8E)
- [Blog](https://cloudchamp.notion.site/How-I-Passed-my-CKA-Exam-55fef633b454438aadc54a7261312ec9)

### [Exam questions](https://skillcertpro.com/product/certified-kubernetes-administrator-cka-exam-questions/)
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

