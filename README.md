# Kubernetes Primer
automatic deployment, scaling and management of containerized applications.

groups containers that make up an application into logical units for easy management and discovery.

## Kubernetes Basic Features
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

## Launch a single node kubernetes cluster
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
