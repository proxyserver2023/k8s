# Kubernetes Primer
1. Automatic Deployment
2. Scaling
3. Management of containerized applications
4. Groups containers that make up an application into logical units for easy management and discovery.
5. K8s is not a mere orchestration system, it eliminates the need for orchestration. 
6. The technical definition of orchestration is execution of a defined workflow. 
   - First Do A -> then B -> then C.
7. In contrast, K8s is a set of independent, composable control processes that continuously drive the current state towards the provided desired state.

## Table of Contents
- Why
- Concepts
- kubernetes basics
- Setup
- Creating a custom cluster from scratch
- Launch a single node kubernetes cluster
- Getting Started with Kubeadm

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

- **Labels** - organize resources
- **Annotation** - decorate resources with custom info
- **Virtualization** - Os level virtualization rather than Hardware level virtualization. containers are isolated from each other and from the host. they have their own filesystems, they can't see each others porcesses, and their computational resources usage can be bounded.
- **Kubernetes Master** - three processes: kube-apiserver, kube-controller-manager, kube-scheduler
- **Kubernetes Non-Master** - two proceses: kubelet, kube-proxy


### Required Fields for `.yaml` file
1. `apiVersion`
2. `kind`
3. `metadata` - Data that helps uniquely identify the object, including a name string, UID, and optional namespace

### Names
All objects in the Kubernetes REST API are unambiguously identified by a Name and a UID.

### Namespaces
K8s supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called namespaces.
```shell
kubectl get namespaces
```

* Output:
```
NAME          STATUS   AGE
------------------------------
default       Active   9m38s
kube-public   Active   9m34s
kube-system   Active   9m37s
```

```shell
# In a namespace
$ kubectl api-resources --namespaced=true

# Not in a namespace
$ kubectl api-resources --namespaced=false
```

### Labels and Selectors
* **Labels** - Labels can be used to organize and to select subsets of objects.
Labels allow for efficient queries and watches and are ideal for use in UIs and CLIs. Non-identifying information should be recorded using annotations.

### Annotations
You can use Kubernetes annotations to attach arbitrary non-identifying metadata to objects. Clients such as tools and libraries can retrieve this metadata.

```json
"metadata": {
  "annotations": {
    "key1" : "value1",
    "key2" : "value2"
  }
}

```

## k8s commands
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

## Getting Started with Kubeadm
### Initialize Master
The first stage of initialising the cluster is to launch the master node. The master is responsible for running the control plane components, etcd and the API server. Clients will communicate to the API to schedule workloads and manage the state of the cluster.

``` bash
kubeadm init --token=102952.1a7dd4cc8d1f4cc5 --kubernetes-version $(kubeadm version -o short)

sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf

```

### Join Cluster

``` shell
kubeadm token list
```

On the Second Node

``` shell
kubeadm join --discovery-token-unsafe-skip-ca-verification --token=102952.1a7dd4cc8d1f4cc5 172.17.0.20:6443
```

`--discovery-token-unsafe-skip-ca-verification` is used to bypass the Discovery token verfication.

### View Nodes

``` shell
kubectl get nodes
```

The Nodes are not ready, cause CNI has not been deployed.

### Deploy CNI
The Container Network Interface (CNI) defines how the different nodes and their workloads should communicate. 

* On Master

``` shell
cat /opt/weave-kube
kubectl apply -f /opt/weave-kube // master
kubectl get pod -n kube-system // master

kubectl run http --image=katacoda/docker-http-server:latest --replicas=1 // master
kubectl get pods // master
docker ps | grep docker-http-server // 2nd node
```

## Deploy Containers using YAML
* Deployment file
``` yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
    name: webapp1
spec:
    replicas: 1
    template:
        metadata:
            labels:
                app: webapp1
        spec:
            containers:
            - name: webapp1
              image: katacoda/docker-http-server:latest
              ports:
              - containerPort: 80
```

* `kubectl create -f deployment.yaml`
* `kubectl get deployment`
* `kubectl describe deployment webapp1`
* Service
  - The Service selects all applications with the label webapp1. As multiple replicas, or instances, are deployed, they will be automatically load balanced based on this common label. The Service makes the application available via a NodePort.

``` yaml
# This is your Editor pane.
apiVersion: v1
kind: Service
metadata:
    name: webapp1-svc
    labels:
        app: webapp1
spec:
    type: NodePort
    ports:
    - port: 80
      nodePort: 30080
    selector:
        app: webapp1
```
* `kubectl create -f service.yaml`
* `kubectl get svc`
* `kubectl describe svc webapp1-svc`
* `curl host01:30080`
### Scale Deployment
* `kubectl apply -f deployment.yaml`
* `kubectl get deployment`
* `kubectl get pods`
* `curl host01:30080`
# Tutorial
## Hello Minikube
1. create a deployment that manages a pod. the pod runs a container based on the provided Docker image.
``` shell
kubectl create deployment \
	hello-node \ 
	--image=gcr.io/hello-minikube-zero-install/hello-node
```

2. view the deployment
``` shell
kubectl get deployments
```

3. view the pod
``` shell
kubectl get pods
```

4. view the `kubectl` configuration
``` shell
kubectl config view
```
Output:

``` yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/alamin/.minikube/ca.crt
    server: https://192.168.99.100:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/alamin/.minikube/client.crt
    client-key: /home/alamin/.minikube/client.key
```

5. Create a service
By default, the Pod is only accessible by its internal IP address within the kubernetes cluster. To make the `hello-node` container accessible from outside the k8s virtual network, you have to expose the pod as kubernetes `Service`.

6. Expose the pod to the public internet using the `kubectl expose` command:
``` shell
kubectl expose deployment hello-node --type=LoadBalancer --port=8080
```
7. view the services you just created:
``` shell
kubectl get services
```
Output:
``` shell
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
hello-node   LoadBalancer   10.105.34.93   <pending>     8080:30189/TCP   50s
kubernetes   ClusterIP      10.96.0.1      <none>        443/TCP          1d
```

8. `minikube service hello-world`
9. list the currently supported addons:
`minikube addons list`
Output:

``` 
minikube addons list 
- addon-manager: enabled
- coredns: enabled
- dashboard: enabled
- default-storageclass: enabled
- efk: disabled
- freshpod: disabled
- heapster: disabled
- ingress: disabled
- kube-dns: disabled
- metrics-server: disabled
- nvidia-driver-installer: disabled
- nvidia-gpu-device-plugin: disabled
- registry: disabled
- registry-creds: disabled
- storage-provisioner: enabled
```

10. to enable an addon: `minikube addons enable <addon_name>`
11. to disable an addon: `minikube addons disable <addon_name>`
12. view the pod and service you just created:
`kubectl get pod, svc -n kube-system`
Output:

``` shell
NAME                                        READY   STATUS    RESTARTS   AGE
pod/coredns-c4cffd6dc-hnmks                 1/1     Running   1          1d
pod/etcd-minikube                           1/1     Running   0          42m
pod/kube-addon-manager-minikube             1/1     Running   1          1d
pod/kube-apiserver-minikube                 1/1     Running   0          42m
pod/kube-controller-manager-minikube        1/1     Running   0          42m
pod/kube-dns-86f4d74b45-cr7p7               3/3     Running   5          1d
pod/kube-proxy-ww2jz                        1/1     Running   0          41m
pod/kube-scheduler-minikube                 1/1     Running   1          1d
pod/kubernetes-dashboard-6f4cfc5d87-cmq24   1/1     Running   3          1d
pod/storage-provisioner                     1/1     Running   2          1d

NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
service/kube-dns               ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP   1d
service/kubernetes-dashboard   ClusterIP   10.108.163.109   <none>        80/TCP          1d
```
13. clean up
``` shell
kubectl delete service hello-node
kubectl delete deployment hello-node
```

14. stop the minikube VM
``` shell
minikube stop
```

15. Optionally delete the Minikube VM:
``` shell
minikube delete
```


## Deploying an Application

``` shell
kubectl run kubernete-bootcamp \
	--image=gcr.io/google-samples/kubernetes-bootcamp:v1 \
	--port=8080
```
1. searches for a suitable node where an instance of the application could be run.
2. scheduled the application to run on that Node
3. configured the cluster to reschedule the instance on a new Node when needed.

``` shell
kubectl get deployments
```

Pods that are running inside k8s are running on a private, isolated network. By default, they are visible from other pods and services within the same k8s cluster, but not outside that network.

The kubectl command can create a proxy that will forward communications into the cluster-wide, private network.

``` shell
kubectl proxy
```

### Viewing Pods and Nodes
The containers in a Pod share -
1. an IP Address
2. an Port Space
3. always co-located and co-scheduled.
4. and run in a shared context on the same node.

Every k8s node runs at least -
1. `kubelet` a process responsible for communication between the k8s master and the node. it manages pods and containers of that node.
2. A container RT like (docker, rkt) responsible for pulling the container image from a registry, unpacking the container and running the application.

### Troubleshooting with `kubectl`
1. `kubectl get` - list resources
2. `kubelet describe` - show detailed information about a resource.
3. `kubectl logs` - print the logs from a container in a pod.
4. `kubectl exec` - execute a command on a container in a pod.

### Exploring your application
1. get pod name

``` shell
export POD_NAME=$(kubectl get pods -o go-template --template \
	'{{range .items}} \
		{{.metadata.name}} \
		{{"\n"}}
	 {{end}}'
	)
echo Name of the POD: -> $POD_NAME
```

2. output of our application run a curl
``` shell
curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/
```
The url is the route to the API of the Pod.

3. View the container logs
``` shell
kubectl logs $POD_NAME
```

4. execute a command inside a pod's container
``` shell
kubectl exec $POD_NAME env
```
Since we have only single container inside that pod, so mentioning the name won't be necessary.

5. start a bash session in a pod's container
``` shell
kubectl exec -ti $POD_NAME bash
```

### Service to expose your app
A ReplicaSet might then dynamically drive the cluster back to desired state via creation of new Pods to keep your application running.

A Service in Kubernetes is an abstraction which defines a logical set of Pods and a policy by which to access them. 

The set of pods targeted by a service is usually determined by a LabelSelector.

Service can be exposed by specifying a `type` in the ServiceSpec.
- ClusterIP (default)
   - Exposes the Service on an internal IP in the cluster. This type makes the Service only reachable from within the cluster.
- NodePort
  - Exposes the Service on the same port of each selected Node in the cluster using NAT. Makes a Service accessible from outside the cluster using <NodeIP>:<NodePort>. Superset of ClusterIP.
- LoadBalancer
  - Creates an external  load balancer in the current cloud (if supported) and assigns a fixed, external IP to the Service. Superset of NodePort
- ExternalName
  - Exposes the Service using an arbitrary name (specified by externalName in the spec) by returning a CNAME record with the name. No proxy is used.

### Exposing your app commands
``` shell
kubectl get pods -l run=kubernetes-bootcamp
kubectl get services -l run=kubernetes-bootcamp
export POD_NAME=$(kubectl get pods -o go-template --template \
	'{{range .items}} \
		{{.metadata.name}} \
		{{"\n"}} \
	{{end}}' \
)
echo Name of the Pod: $POD_NAME

kubectl label pod $POD_NAME app=v1
kubectl describe pods $POD_NAME
kubectl get pods -l app=v1

kubectl delete service -l run=kubernetes-bootcamp
kubectl get services
curl $(minikube ip):$NODE_PORT

kubectl exec -it $POD_NAME curl localhost:8080
```

### Performing a Rolling Update
1. Perform a rolling update using `kubectl`.
2. Users expect applications to be available all the time and developers are expected to deploy new versions of them several times a day.
3. In Kubernetes this is done with rolling updates. 
4. Rolling updates allow Deployments' update to take place with zero downtime by incrementally updating Pods instances with new ones.
5. The new Pods will be scheduled on Nodes with available resources.
6. get deployments
``` shell
kubectl get deployments
```
7. To list the running pods use the `get pods` command:
``` shell
kubectl get pods
```
8. to view the current image version of the app: run a `describe` command.
``` shell
kubectl describe pods
```
9. Use the `set image` command followed by the deployment name and the new image version:
``` shell
kubectl set image deployments/kubernetes-bootcamp \
	kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
```
10. Check the status of the old pod terminating and new pod creating

``` shell
kubectl get pods
```
Output:

``` shell
NAME                                   READY     STATUS        RESTARTS   AGE
kubernetes-bootcamp-5c69669756-4h999   1/1       Terminating   0          9m
kubernetes-bootcamp-5c69669756-jksvb   1/1       Terminating   0          9m
kubernetes-bootcamp-5c69669756-pt8xq   1/1       Terminating   0          9m
kubernetes-bootcamp-5c69669756-t96bw   1/1       Terminating   0          9m
kubernetes-bootcamp-7799cbcb86-7bddk   1/1       Running       0          18s
kubernetes-bootcamp-7799cbcb86-9t6sg   1/1       Running       0          18s
kubernetes-bootcamp-7799cbcb86-fwmlv   1/1       Running       0          16s
kubernetes-bootcamp-7799cbcb86-jg98m   1/1       Running       0          16s
```
11. Let's check the app is running. to get the describe 
``` shell
kubectl describe services/kubernetes-bootcamp
```
12. create an environment variable called `NODE_PORT` that has the value of the Node Port assigned.
``` shell
export NODE_PORT=$(
	kubectl get services/kubernetes-bootcamp \
	-o go-template='{{(index .spec.ports 0).nodePort}}'
)
echo NODE_PORT=$NODE_PORT
```
13. curl to the exposed ip and port
``` shell
curl $(minikube ip):$NODE_PORT
```
14. the update can be confirmed by running a roll-out status

``` shell
kubectl rollout status deployments/kubernetes-bootcamp
```
