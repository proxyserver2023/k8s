# Kubernetes Primer
automatic deployment, scaling and management of containerized applications.

groups containers that make up an application into logical units for easy management and discovery.

K8s is not a mere orchestration system, it eliminates the need for orchestration. 

The technical definition of orchestrationis is execution of  a defined workflow. First Do A -> then B -> then C.

In contrast, K8s is a set of independent, composable control processes that continuously drive the current state towards the provided desired state.

## Table of Contents
- Why
- Concepts
- kubernetes basics
- Setup
- Creating a custom cluster from scratch
- Launch a single node kubernetes cluster

## Why - k8s Basic Features
1. Service Discovery
   - kubernetes gives containers their own IP addresses and a single DNS name for a set of containers and can load-balance across them
2. Automatic Binpacking
   - automatically places containers based on their resource requirements and other containers, while not sacrificing availability.
3. Storage Orchestration
   - local storage / cloud (GCP/AWS) / network storage such as NFS, iSCSI, Gluster, Ceph, Cinder, Flocker
4. Self-healing
   - restarts containers that fail
   - replaces and reschedules containers when nodes die,
   - kills containers that dont respond to your user-defined health check, and doesn't advertise them until they are ready to serve.
5. Automated rollouts and rollbacks
   - progressively rolls out changes to your application, or configuration, while monitoring application health.
   - all instances are not kill at same time.
   - if something goes wrong, kubernetes will rollback the change for you.
6. secret and configuration management 
   - deploy and update secrets and application configuration
   - w/o rebuilding your image and w/o exposing secrets in your stack configuration
7. batch execution - batch and CI workloads, replacing containers that fail, if desired.
8. horizontal scaling

## Concepts
It orchestrates computing, networking, and storage infrastructure on behalf of user workloads

- **Labels** organize resources
- **Annotation** decorate resources with custom info

Os level virtualization rather than Hardware level virtualization. containers are isolated from each other and from the host. they have their own filesystems, they can't see each others porcesses, and their computational resources usage can be bounded.

**Kubernetes Master** - three processes: kube-apiserver, kube-controller-manager, kube-scheduler

**Kubernetes Non-Master** - two proceses: kubelet, kube-proxy

## k8s basics

``` bash
minikube start
minikube dashboard
kubectl create deployment hello-node --image=gcr.io/hello-minikube-zero-install/hello-node
kubectl get deployments
kubectl get pods
kubectl get events
kubectl config view
kubectl expose deployment hello-node --type=LoadBalancer --port=8080
minikube service hello-node
minikube addons list
minikube addons enable heapster

kubectl get pod,svc -n kube-system
# Outputs
# ---------------------------------------------------------------------------------------
# NAME                                        READY   STATUS              RESTARTS   AGE
# pod/coredns-c4cffd6dc-942td                 1/1     Running             2          34d
# pod/etcd-minikube                           1/1     Running             0          2h
# pod/heapster-7nnvr                          0/1     ContainerCreating   0          3s
# pod/influxdb-grafana-8r4cz                  0/2     ContainerCreating   0          3s
# pod/kube-addon-manager-minikube             1/1     Running             2          34d
# pod/kube-apiserver-minikube                 1/1     Running             0          2h
# pod/kube-controller-manager-minikube        1/1     Running             0          2h
# pod/kube-dns-86f4d74b45-nlhq5               3/3     Running             8          34d
# pod/kube-proxy-8cv5s                        1/1     Running             0          2h
# pod/kube-scheduler-minikube                 1/1     Running             0          2h
# pod/kubernetes-dashboard-6f4cfc5d87-ljdqg   1/1     Running             5          34d
# pod/storage-provisioner                     1/1     Running             5          34d
# 
# NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
# service/heapster               ClusterIP   10.104.225.26    <none>        80/TCP              3s
# service/kube-dns               ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP       34d
# service/kubernetes-dashboard   ClusterIP   10.96.163.75     <none>        80/TCP              34d
# service/monitoring-grafana     NodePort    10.110.218.199   <none>        80:30002/TCP        3s
# service/monitoring-influxdb    ClusterIP   10.108.148.52    <none>        8083/TCP,8086/TCP   3s

minikube addons disable heapster
```

``` bash
# Clean up
kubectl delete service hello-node
kubectl delete deployment hello-node
```

``` bash
minikube stop
minikube delete
```

## Setup
### Local Machine Solutions
* Minikube 
  - single node kubernetes cluster
* Minishift 
  - community version of kubernetes enterprise platform
  - Openshift for local development & testing.
* microk8s
  - single command installation of latest kubernetes release.
* IBM Cloud Private-CE or IBM Cloud Private-CE on Linux Containers
* kubadm-dind
  - multinode (while minikube is single-node) kubernetes cluster. only requires a docker daemon. docker-in-docker to spawn the kubernetes cluster.
* Ubuntu on LXD supports a nine-instance deployment on localhost.


## Creating a custom cluster from scratch
### Designing and Preparing
* Cloud Provider
  * provides an interface for managing TCP Load Balancers, Nodes(instances) and Networking Routes
  * **TCP Load Balancer** - verifies information sent to IP addresses. ensures the data arrives error-free to non-HTTP applications.
* Nodes
  * **x86_64 architecture** - 
* Network
  * container to container - solved by pods
  * pod to pod
  * pod to service - services
  * external to service - services
* Docker Model
* Kubernetes Model
  * all containers can communicate with all other containers w/o NAT
  * all nodes can communicate with all containers (and vice-versa) w/o NAT
  * the IP that a container sees itself as is the same IP that others see it as.
  * In previous, your VM had an IP and could talk to other VMs in your project. 
#### Network
* Network connectivity
  - Kubernetes allocates an IP address to each pod.
  - a block of IPs for kubernetes to use as Pod IPs.
  - allocate a different block of IPs to each node in the cluster as the node is added.
  - a process of one pod should be able to communicate with another pod using the IP of the second pod.
	- overlay network
		- traffic encapsulation (vxlan)
		- might have performance issue
	- w/o overlay
		- configure the underlying network fabric(switches, routers) to be aware of pod IP addresses.
		- comparatively performant
	- network plugin called by k8s
		- CNI Plugin interface
#### Components
1. etcd - inside docker as a docker container
2. docker - outside of docker container as system daemon
3. Kubernetes
   - kubelet - outside of docker container as system daemon
   - kube-proxy - outside of docker container as system daemon
   - kube-apiserver - inside docker as a docker container
   - kube-controller-manager - inside docker as a docker container
   - kube-scheduler - inside docker as a docker container
#### Security Models
1. Access the APIServer using http
   - firewall/security
   - easier
2. using HTTPS
   - https with certs, and credentials for user
   - recommended approach
   - configure certs can be tricky

## Launch a single node k8s cluster
Minikube - single node k8s cluster inside a vm on your laptop

``` bash
minikube version
minikube start

# minikube started a VM, kube cluster is running in that VM
# ----------------------------------------------------------
# starting kube cluster 
# starting VM
# getting VM IP address
# moving files to cluster
# setting up certs
# connecting to cluster
# setting up kubeconfig
# starting cluster components
# kubectl is now configured to use the cluster
# loading cached images from the config file
```
* Cluster Info
  - Details of the cluster and health status can be discovered via 

```bash
kubectl cluster-info
# Outputs
# --------
# Kubernetes master is running at https://192.168.99.100:8443
# CoreDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

``` bash
kubectl get nodes
```

``` bash
kubectl run first-deployment --image=katacoda/docker-http-server --port=80
```

``` bash
kubectl get pods
```

``` bash
export PORT=$(kubectl get svc first-deployment -o go-template='{{range.spec.ports}}{{if .nodePort}}{{.nodePort}}{{"\n"}}{{end}}{{end}}')
echo "Accessing host01:$PORT"
curl host01:$PORT
```

