# Linux Academy - Certified Kubernetes Administrators 

# Core Concepts (19% of the Exam)
## Setting Up The Cluster 

Initializing the master node with `kubeadm`. Linux Academy nodes come prepacked with `kubeadm` so follow 
this [link](https://kubernetes.io/docs/setup/independent/install-kubeadm/) for the official documentation. 

`kubeadm init --pod-network-cidr=10.244.0.0/16`

### Setup Pod Networking 

For this course we will be applying flannel to manage our networking.

`kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`

```
cloud_user@ionshipman1c:~$ sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.extensions/kube-flannel-ds-amd64 created
daemonset.extensions/kube-flannel-ds-arm64 created
daemonset.extensions/kube-flannel-ds-arm created
daemonset.extensions/kube-flannel-ds-ppc64le created
daemonset.extensions/kube-flannel-ds-s390x created 
```

```
cloud_user@ionshipman1c:~$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                                   READY   STATUS    RESTARTS   AGE
kube-system   coredns-576cbf47c7-b8tcp                               1/1     Running   0          17m
kube-system   coredns-576cbf47c7-wbslz                               1/1     Running   0          17m
kube-system   etcd-ionshipman1c.mylabserver.com                      1/1     Running   0          17m
kube-system   kube-apiserver-ionshipman1c.mylabserver.com            1/1     Running   0          16m
kube-system   kube-controller-manager-ionshipman1c.mylabserver.com   1/1     Running   0          16m
kube-system   kube-flannel-ds-amd64-wrrp7                            1/1     Running   0          6m59s
kube-system   kube-proxy-s9h9r                                       1/1     Running   0          17m
kube-system   kube-scheduler-ionshipman1c.mylabserver.com            1/1     Running   0          17m
```

### Joining Machines to the Cluster

Join machines to the cluster by running the following command:
`kubeadm join 172.31.44.139:6443 --token 3awyml.20bmlx2eti0hngs3 --discovery-token-ca-cert-hash sha256:baceefabb399fd37499b1cfe34e7f50faa5e5ddd3319222f14fbd2e4a9823d19` 

```
cloud_user@ionshipman1c:~$ kubectl get nodes
NAME                           STATUS     ROLES    AGE    VERSION
ionshipman1c.mylabserver.com   Ready      master   22m    v1.12.2
ionshipman2c.mylabserver.com   Ready      <none>   105s   v1.12.2
ionshipman3c.mylabserver.com   NotReady   <none>   14s    v1.12.2
``` 

Upon a node joining the cluster, Kubernetes will spin up additional pods on the new nodes by utilizing daemonsets. In this case, the `kube-proxy` and `kube-flannel-ds-amd64` pods. 

```
cloud_user@ionshipman1c:~$ kubectl get po --all-namespaces
NAMESPACE     NAME                                                   READY   STATUS    RESTARTS   AGE
kube-system   coredns-576cbf47c7-b8tcp                               1/1     Running   0          24m
kube-system   coredns-576cbf47c7-wbslz                               1/1     Running   0          24m
kube-system   etcd-ionshipman1c.mylabserver.com                      1/1     Running   0          23m
kube-system   kube-apiserver-ionshipman1c.mylabserver.com            1/1     Running   0          23m
kube-system   kube-controller-manager-ionshipman1c.mylabserver.com   1/1     Running   0          23m
kube-system   kube-flannel-ds-amd64-6528m                            1/1     Running   0          103s
kube-system   kube-flannel-ds-amd64-dft79                            1/1     Running   0          3m14s
kube-system   kube-flannel-ds-amd64-wrrp7                            1/1     Running   0          13m
kube-system   kube-proxy-9bpqs                                       1/1     Running   0          3m14s
kube-system   kube-proxy-lgqx9                                       1/1     Running   0          103s
kube-system   kube-proxy-s9h9r                                       1/1     Running   0          24m
kube-system   kube-scheduler-ionshipman1c.mylabserver.com            1/1     Running   0          23m
```

## Architecture and Generic Installation 

View the kubernetes architecture in the `linuxacedemy-kubernetesadmin-archdiagrams-1_516737832.pdf`. 

### Raw Kubernetes Install -- CentOS 

```
Kubernetes Ports: 
Master Node(s):
    TCP 6443*       Kubernetes API server 
    TCP 2379-2380   etcd server client API 
    TCP 10250       Kubelet API
    TCP 10251       kube-scheduler
    TCP 10252       kube-controller-manager
    TCP 10255       Read-Only kubelet API  

Worker Node(s): 
    TCP 10250       Kubelet API 
    TCP 10255       Read-Only Kubelet API 
    TCP 30000-32767 NodePort Services 
```
1. Disable swap on the CentOS  server 
    - `sudo swapoff -a` 
    - `sudo vim /etc/fstab` and comment out the swap entry. 

2. Install docker 
    - `yum -y install docker` 
    - `systemctl enable docker`ß
    - `systemctl start docker` 

3. Add the kubernetes repo 
```
    cat << EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
    [kubernetes]    
    name=Kubernetes
    baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    EOF
```

4. Turn off selinux 
    - `setenforce 0` 
    - `vim /etc/selinux/config` change `SELINUX=enforcing` to `SELINUX=permissive` 

5. Install Kubernetes 
```
    sudo yum install -y kubelet kubeadm kubectl
    sudo systemctl enable kubelet
    sudo systemctl start kubelet
```

6. Configure sysctl for Kubernetes networking 
```
    cat << EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
    sudo sysctl --system 
```
7. Initialize the Kube Master 
8. Install flannel networking. 

### API Primitives

- Persustebt entities in Kubernetes System. 
- Use the API to represent the state of the cluster. 
- Describe: 
    - What aplications are running. 
    - Which nodes those applications are running on. 
    - Policies around those applications. 
- Kubernetes Objects are "records of intent."  

Object Spec: 
- Provided to Kubernetes by the amdin. 
- Describes desired state of objects/

Object Status: 
- Provided by Kubernetes 
- Describes the actual state of the object.  

All objects will need the following: 

apiVersion
kind 
metadata 
spec 

ex. 

`Kubernetes Yaml` 

```
apiVersion: v1 
kind: Pod 
metadata: 
  name: busybox 
spec: 
  containers: 
  - name: busybox 
    image: busybox 
    command: 
      - sleep 
      - "3600"
```

Common Objects: 

Nodes 
Pods 
Deployments 
Services 
ConfigMaps

Names and UIDs 

Names: 
- All objects have a unique name 
- Client proveded. 
- Can be reused 
- Maximum length of 253 characters.  
- Lower case alphanumeric characters. 
- `-` and `.` allowed 

UIDs: 
- All objects have a unique UID> 
- Generated by Kubernetes. 
- Spatially and temporarily unique.  

Namespaces: 
- Multiple virtual clusters back by the same virtual cluster. 
- Generally for large deployments 
- Scope for names 
- Easy way to divide cluster resources. 
- Allows for multiple teams of users 
- Allows for resource quotas. 
- Special "kube-system" namespace. 
  - used to differentiate system pods from user pods. 

Nodes: 
- Might be a VM or physical machine 
- Services necessary to run pods 
- Managed by the master 
- Services necessary: 
  - Container runtime 
  - Kubelet 
  - Kube-proxy 
- Not inherently create dby Kubernetes, but by the cloud provider. (ex AWS, GCP, etc)
- Kubernetes checks the node for validity. 

Cloud Controller Managers: 
- Route controller (gce clusters only) - Configuring networking
- Service Controller - Create, update and delete events and setups ELBS etc.
- PersistentVolumeLabels controller - Applies labels for AWS and GCP volumes.  

Node Controller: 
- Assigns CIDR blocks to a newly registered node. 
- Keeps track of the nodes. 
- Monitors the node heakth. 
- Evicts pods from unhealthy nodes. 
- Can tain nodes based on current conditions in more recent versions.  

### Kubernetes Services & Network Primitives 

Kubernetes Services: 
- Underlying architecture 
- Pod -- Simplest Kubernetes object, represents one or more containers running on a single node. 
- Ephemeral, disposable and replaceable -- Stateless 
- "Cattle vs. Pets" 
- Usually Managed via Deployments  

Deployment specifications 
- Image 
- Number of replicas 

Services --> Deployments 
- Particular port or IP address 

- Running the application pods. 
- How you set up a service depends on your networking configuration and how you will handle load balancing and port forwarding. 
- If you use the pod network IP address method, then a deployment gets assigned a single IP address -- even if there are multiple replicas of that pod. 
- The kubernetes service (using kube-proxy on the node) redirects traffic.  

Imperative 
- `kubectl run nginx --image=nginx` or `kubectl create secret generic hello-kube-conf --from-file=hello-kube.conf`

Declarative 
```
    apiVersion: apps/v1 
    kind: Deployment 
    metadata: 
      name: nginx 
      labels: 
        app: nginx 
    spec: 
      replicas: 3 
      selector: 
        matchLabels: 
          app: nginx
```

Points to remember: 
- Containers are run in Pods. Pods are the simplest Kubernetes object representing an application. 
- Pods are (usually) managed by deployments. 
- Services expose deployments. 
- Third parties handle load balancing or port forwarding to those services, though Ingress objects (along with an appropriate ingress controller) are needed to do that work. 

#Installation, Configuration and Validation (12% of Exam)
# Designing a Kubernetes Cluster 

*Minikube* - Recommended method for creating a single node kubernetes deployment on local workstation. 

*Kubeadm* - Deploy a multi-node locally. You'll need to select your own CNI (Cluster Network Interface) 

*Ubuntu* 

*Turnkey Solutions* - Conjure-up Kubernetes with Ubuntu on AWS, Azure, Google Cloud, etc. 

Add-on Solutions - Vendors offer a wide variety of add-ons to k8s. 
CNI - Container Networking Interface: 
    *Calico* - Secure L3 networking and network policy provider.  
    *Canal* - Canal unites Flannel and Calico, providing networking and network policy.
    *Cillium* - L3 network and network policy plugin that can enforce HTTP/API/L7 policies transparently. Both routing and overlay/encapsulation mode are supported.
    *CNI-Genie* - Enables Kubernetes to seamless connect to a choice of CNI plugins, such as Calico, Canal, Flannel, Romana, or Weave. 
    *Contiv* - Provides configurable networking (native L3 using BGP, overlay using vxlan, ckassic L2, and Cisco-SDN/ACI) for various use cases and a rich policy framework. Fully open sourced and the install provides a kubeadm and non-kubeadm approach. 
    *Flannel and Multus* - Over netowrk provider that can be used with Kubernetes. Multus is a plugin for multiple network support in Kubernetes to support all CNI plugins in addition to SRIOV, DPDK, OVS-DPDK and VPP based workloads in Kubernetes.

Other Solutions    
*NSX-T Containers Plugin-in (NCP)* - provides integration between VMware NSX-T and container orchestrators such as Kubernetes. 
Nuage, Romana, and Weave Net.

# Excercise: Explore the Sandbox 

1. Examine the current status of your cluster. Are all the nodes ready? How do you know?
```
cloud_user@ionshipman1c:~/kubernetes$ kubectl get nodes
NAME                           STATUS   ROLES    AGE   VERSION
ionshipman1c.mylabserver.com   Ready    master   18d   v1.12.2
ionshipman2c.mylabserver.com   Ready    <none>   18d   v1.12.2
ionshipman3c.mylabserver.com   Ready    <none>   18d   v1.12.2
```

2. Are there any pods running on node 2 of your cluster? How can you tell?
` kubectl describe node <node-name>` 

```
cloud_user@ionshipman1c:~/kubernetes$ kubectl describe node ionshipman3c.mylabserver.com
Name:               ionshipman3c.mylabserver.com
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/hostname=ionshipman3c.mylabserver.com
Annotations:        flannel.alpha.coreos.com/backend-data: {"VtepMAC":"6e:85:bc:c8:6e:06"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 172.31.47.56
                    kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Fri, 16 Nov 2018 17:42:27 +0000
Taints:             <none>
Unschedulable:      false
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  OutOfDisk        False   Wed, 05 Dec 2018 14:19:34 +0000   Fri, 16 Nov 2018 17:42:27 +0000   KubeletHasSufficientDisk     kubelet has sufficient disk space available
  MemoryPressure   False   Wed, 05 Dec 2018 14:19:34 +0000   Fri, 16 Nov 2018 17:42:27 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Wed, 05 Dec 2018 14:19:34 +0000   Fri, 16 Nov 2018 17:42:27 +0000   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Wed, 05 Dec 2018 14:19:34 +0000   Fri, 16 Nov 2018 17:42:27 +0000   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Wed, 05 Dec 2018 14:19:34 +0000   Fri, 16 Nov 2018 17:42:47 +0000   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  172.31.47.56
  Hostname:    ionshipman3c.mylabserver.com
Capacity:
 cpu:                1
 ephemeral-storage:  20263528Ki
 hugepages-2Mi:      0
 memory:             2046684Ki
 pods:               110
Allocatable:
 cpu:                1
 ephemeral-storage:  18674867374
 hugepages-2Mi:      0
 memory:             1944284Ki
 pods:               110
System Info:
 Machine ID:                 e156aebfbcac49b4bed31684a6b812cb
 System UUID:                EC23F29C-9AB0-AA17-3EF6-9F5CD0F06E39
 Boot ID:                    0ee61a38-dfef-4e8d-adbe-466901d3afec
 Kernel Version:             4.4.0-1072-aws
 OS Image:                   Ubuntu 16.04.5 LTS
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://17.3.2
 Kubelet Version:            v1.12.2
 Kube-Proxy Version:         v1.12.2
PodCIDR:                     10.244.2.0/24
Non-terminated Pods:         (3 in total)
  Namespace                  Name                           CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ---------                  ----                           ------------  ----------  ---------------  -------------
  default                    nginx                          0 (0%)        0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-flannel-ds-amd64-6528m    100m (10%)    100m (10%)  50Mi (2%)        50Mi (2%)
  kube-system                kube-proxy-lgqx9               0 (0%)        0 (0%)      0 (0%)           0 (0%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource  Requests    Limits
  --------  --------    ------
  cpu       100m (10%)  100m (10%)
  memory    50Mi (2%)   50Mi (2%)
Events:     <none>
```
3. Is the master node low on memory currently? How can you tell?
`kubectl describe node <node-name>`
```
... 
... 
...
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource  Requests    Limits
  --------  --------    ------
  cpu       100m (10%)  100m (10%)
  memory    50Mi (2%)   50Mi (2%)
Events:     <none>
```
4. What pods are running in the kube-system namespace? What command did you use to find out?
```
cloud_user@ionshipman1c:~/kubernetes$ kubectl get pods -n kube-system
NAME                                                   READY   STATUS    RESTARTS   AGE
coredns-576cbf47c7-b8tcp                               1/1     Running   3          18d
coredns-576cbf47c7-wbslz                               1/1     Running   3          18d
etcd-ionshipman1c.mylabserver.com                      1/1     Running   3          18d
kube-apiserver-ionshipman1c.mylabserver.com            1/1     Running   3          18d
kube-controller-manager-ionshipman1c.mylabserver.com   1/1     Running   3          18d
kube-flannel-ds-amd64-6528m                            1/1     Running   3          18d
kube-flannel-ds-amd64-dft79                            1/1     Running   3          18d
kube-flannel-ds-amd64-wrrp7                            1/1     Running   3          18d
kube-proxy-9bpqs                                       1/1     Running   3          18d
kube-proxy-lgqx9                                       1/1     Running   3          18d
kube-proxy-s9h9r                                       1/1     Running   3          18d
kube-scheduler-ionshipman1c.mylabserver.com            1/1     Running   3          18d
```

# Hardware and Underlying Infrastructure 

- Nodes, including the master, can be physical or virtual machines running kubernetes components and a container manager such as docker or rocket. 
- A pod networking application such as Flanner is needed to allow the pods to communicate, and makes use of an overla network (by default it's vxlans) to provide that service.  

** For the exam in K8s lab is made of of 3 virtual machines. Kubeadm to bootstrap the master and ran a join command on each node to provision them. 

# Securing Cluster Communications  

Cluster communications cover communications to the API server, control-plane communications inside the cluster, and can even include pod-to-pod communications. 
Everything in Kubernetes goes through the API server. 

Default encription in K8s is TLS. 
Most installation methods handle the certificate creation. (Kubeadm) 
No matter how you've installed kubernetes, some components and installation methods may enable local ports over HTTP. Double check the settings of these components and address them.  

Anything that conencts to the API, including nodes, proxies, the scheduler, volume plugins and users should pass an authentication check. 
Once authenticated, every API call should pass an authorization check.  

## Role-Based Access Control (RBAC) 

Certain roles perform specific actions in the cluster. 
Simple roles might be fine for small clusters. 
If a user doesn't have rights to perform an action but they have access to perform a composite action that includes it, the user WILL be able to indirectly create objects. 
ex. If a user can't create a pod, but can create a deployments, the pod will be launched.  

## Securing the Kubelet 

Secure the kubelet on each node. The Kubelets expose HTTPS endpoints which give access to both data and actions on the nodes. By default, these are open. 
To secure those endpoints, you can enable Kubelet Authenticaton and Authorization by starting it with an `--anonymous-auth=false` flag and assigning it an appropriate x509 client certificate in its configurations.  

## Securing the Network 

Network policies restric access to the network for a particular namespace. This allow developers to restrict which pods in other namespaces can access pods and ports within the current namespace. THe pod networking CNI must respect these policies. 

Users can also be assigned quotors or limit ranges. 

Use plug-ins for more advanced functionality. 

## Vulnerabilities 

Kubernetes makes extensive use of etc for storing configuration and secrets. It acts as the key/value store for the whole cluster.  

Gaining write access to etcd is very much like gaining root on the whole cluster, and even read access can be used by attackers to cause damage. 

Strong credentials on your etc server or cluster is a must. 

Isolate the etc resources behind a firewall that only allows requests from the API servers. 

Audit logging is also critical, it records actions taken by the API for later analysis in the event of an attack. Enable audit logging and archie the audit file on a secure server.  

Rotate your infrastructure credentials frequently. Smaller lifetime windows for secrets and credentials create bigger problems for attackers attempting to use it. Set these up with short lifetime cycle and automate the rotation. 

## Third Party Integrations 

Always review third party integrations before enabling them. Integrations to K8s can change how secure your cluster is. 
Add-ons might be nothing more than a pod in the cluster - Don't allow them into the `kube-system` namespace. 

# Making Kubernetes Highly Available 

## The Process 

- Create the reliable nodes that will form our cluster. 
- Setup a redundant and reliable storage service with a multinode deployment of etcd. 
- start replicated and load balanced Kubernetes API servers. 
- Set up a master-elected kubernetes scheduler and controller manager daemons. 

Everything that needs the API server on any other service on the master foes through the load balancer, including the worker nodes.  

## Step One - Make the master node reliable. 
- Ensure that the services automatically restart if they fail 
- Kubelet already does this, if kubelet goes down, we need something to restart it. Monit on Debian or systemctl systemd systems can handle that.  

## Step Two - Storage Layer 
- Clustered etc already replicates the storage to all master instances in your cluster. 
- To lose data, all three nodes would need to have their disks fail at the same time. 
- Probability of this occurrence is low, running a replicated etc cluster is reliable enough. 
- Additional reliability by icnreasing the size of the cluster from 3 to 5 nodes.  

## Step Three - Replicated API Services 
-  Create the initial log file so that Docker will mount a file instead of a directory:  
  - `touch /var/log/kube-apiserver.log`
- Create a `/src/kubernetes/` directory on each node which should include: 
  - basic_auth.csv - basic auth user and password 
  - ca.crt - Certificate Authority cert 
  - known_tokens.csv - tokens that entities (e.g the kubelet) can use to talk t the apiserver 
  - kubecfg.crt - Client certificate, public key 
  - kubecfg.key - Client certificate, private key 
  - server.cert - Server certificate, public key 
  - server.key - Server certificate, private key  

- Create manually or copy from master node on a working cluster
- Copy the `kube-apiserver.yml` into `/etc/kubernetes/manifests/` on each of our master nodes
- Kubelet monitors this directory, and will automatically create an instance of the kube-apiserver container using the pod definition specified in the file. 

- If network load balance ris setup, access the cluster using the VIP to verify traffic balancing between the apiserver instances. 
- External users (kubectl command line interface, continuous build pipelines, etc) remember to configure them to talk to the external load balancers' IP address.

## Step Four - Controller / Scheduler Daemons 

- Allow our state to change
- controller managers and scheduler processes must not modify the cluster's state simultaneously, so we must use a lease-lock. 
- Each scheduler and controller manager can be launched with a `--leader-elect` flag  
- Scheduler and controller-manager can be configured to talk to the API server on the same node (127.0.0.1) or the load balanced IP address. 
- Scheduler and controller-manager will complete the leader election process mentioned before when using the `--leader-elect` flag. 
- In case of a failure accessing the API server, the elected leader will not be able to renew the lease causing a new leader to be elected. 
- Especially relevant when configuring the scheduler and controller-manaer to access the API server via 127.0.0.1 and the API server on the same node is unavailable.

## Installing Configuration Files 

- Create empty log files on each node, so Docker will mount the files and not make new directories: 
  - `touch /var/log/kube-scheduler.log`
  - `touch /var/log/kube-controller-manager.log` 

- Set up the descriptions of the scheduler and controller manager pods on each node by copying `kube-scheduler.yaml` and `kube-manager.yaml` into the `/etc/kubernetes/manifests/` directory.

# Validating Nodes and the Cluster 

## End to End Validations 
- Provides a mechanism to test end-to-end behavior of the system. It's the last signal to ensure end user operations match speficfications and is primarily a developer tool. 
- Difficul to run against "any" deployment -- many specific tests for cloud providers. 
  - Ubuntu has its own Juju-deployed tests 
  - GCE Has its own 

- Kubetest Suite 
  - ideal for GCE or AWS users
  - Build 
  - Stage 
  - Extract 
  - Bring up the cluster 
  - Test
  - Dump logs 
  - Tear down

## Validating Nodes and the Cluster 

Validating the nodes and cluster 

`kubectl get nodes` 

```
cloud_user@ionshipman1c:~$ kubectl get nodes
NAME                           STATUS   ROLES    AGE   VERSION
ionshipman1c.mylabserver.com   Ready    master   24d   v1.12.2
ionshipman2c.mylabserver.com   Ready    <none>   24d   v1.12.2
ionshipman3c.mylabserver.com   Ready    <none>   24d   v1.12.2
```

`kubectl describe nodes ionshipman1c.mylabserver.com` 

Outputs the nodes status - CPU / Disk Usage, taints etc. 

Additonally we can SSH into the node to verify the status 

`ps aux | grep kube` to verify kue services are up and running. 

# Application Lifecycle Management (8%) 

## Deployments, Rolling Updates and Rollbacks 

Creating a deployment and checking the status of the deployment. 

```
cloud_user@ionshipman1c:~/kubernetes$ kubectl create -f nginx-deployment.yml
deployment.apps/nginx-deployment created
cloud_user@ionshipman1c:~/kubernetes$ kubectl get po
NAME                                READY   STATUS              RESTARTS   AGE
nginx                               1/1     Running             3          6d3h
nginx-deployment-5c689d88bb-7fhdt   0/1     ContainerCreating   0          5s
nginx-deployment-5c689d88bb-cjkrw   0/1     ContainerCreating   0          5s
cloud_user@ionshipman1c:~/kubernetes$ kubectl get deployments
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2         2         2            2           22s
cloud_user@ionshipman1c:~/kubernetes$ kubectl describe deployment nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Tue, 11 Dec 2018 17:50:10 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.7.9
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-5c689d88bb (2/2 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  45s   deployment-controller  Scaled up replica set nginx-deployment-5c689d88bb to 2
``` 

Returning the yaml 
`kubectl get deployment nginx-deployment -o yaml` 

## Rolling updating images 
1. Via command line 
```
cloud_user@ionshipman1c:~/kubernetes$ kubectl set image deployment/nginx-deployment nginx=nginx:1.8
deployment.extensions/nginx-deployment image updated
cloud_user@ionshipman1c:~/kubernetes$ kubectl rollout status deployment/nginx-deployment
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deployment" successfully rolled out 
kubectl describe deployment nginx-deployment # Output and verify new image
```

2. Via yaml 
Update then `kubectl apply -f nginx-deployment.yml` 

## Checking the rollout history for debugging

```
cloud_user@ionshipman1c:~/kubernetes$ kubectl rollout history deployment/nginx-deployment --revision=3
deployment.extensions/nginx-deployment with revision #3
Pod Template:
  Labels:	app=nginx
	pod-template-hash=6987cdb55b
  Containers:
   nginx:
    Image:	nginx:1.9.1
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

## Rolling back a Deployment

```
cloud_user@ionshipman1c:~/kubernetes$ kubectl rollout undo deployment/nginx-deployment --to-revision=2
deployment.extensions/nginx-deployment
cloud_user@ionshipman1c:~/kubernetes$ kubectl apply -f nginx-deployment.yml
deployment.apps/nginx-deployment configured
cloud_user@ionshipman1c:~/kubernetes$ kubectl rollout status deployment nginx-deployment
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deployment" successfully rolled out
cloud_user@ionshipman1c:~/kubernetes$
```

## Excercise: Deployments 

## Excercise

1. Create the deployment.

2. Which nodes are the pods running on. How can you tell?

3. Update the deployment to use the 1.8 version of the nginx container and roll it out.

4. Update the deployment to use the 1.9.1 version of the nginx container and roll it out.

5. Roll back the deployment to the 1.8 version of the container.

6. Remove the deployment

```
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

## Solution 
```
1. Create the yaml file and name it something. I chose nginx-deployment.yaml. Create the deployment object by calling kubectl create -f nginx-deployment.yaml.

2. You can find this answer by doing a kubectl describe deployment nginx-deployment.

3. There are many ways to get this:

kubectl get pods -l app=nginx -o wide gives you the results in one step and uses a label selector.
Or, you could:
kubectl describe deployment nginx-deployment to get the pod information about the deployment and, using that,
kubectl get pods name-of-pods -o wide
4. There are many ways. Here are two:

kubectl set image deployment nginx-deployment nginx=nginx:1.8. This will work just fine but is not the preferred method because now the yaml is inconsistent with what you've got running in the cluster. Anyone coming across your yaml will assume it's what is up and running and it isn't.
Update the line in the yaml to the 1.8 version of the image, and apply the changes with kubectl apply -f nginx-deployment.yaml
5. Same as above. Don't forget you can watch the status of the rollout with the command kubectl rollout status deployment nginx-deployment.

6. kubectl rollout undo deployment nginx-deployment will undo the previous rollout, or if you want to go to a specific point in history, you can view the history with kubectl rollout history deployment nginx-deployment and roll back to a specific state with kubectl rollout history deployment nginx-deployment --revision=x.
```
### Finding the Nodes pods are running on 

```
cloud_user@ionshipman1c:~/kubernetes$ kubectl get pods -l app=nginx -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP            NODE                           NOMINATED NODE
nginx-deployment-6987cdb55b-ch94h   1/1     Running   0          18m   10.244.2.13   ionshipman3c.mylabserver.com   <none>
nginx-deployment-6987cdb55b-z7d42   1/1     Running   0          18m   10.244.1.9    ionshipman2c.mylabserver.com   <none>
```

## How Kubernetes Configures Applications 

Use config maps

1. from command line 
```
cloud_user@ionshipman1c:~/kubernetes$ kubectl create configmap my-map --from-literal=school=LinuxAcademy
configmap/my-map created
cloud_user@ionshipman1c:~/kubernetes$ kubectl describe configmaps my-map
Name:         my-map
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
school:
----
LinuxAcademy
Events:  <none>
```

Returning the configmap 

```
cloud_user@ionshipman1c:~/kubernetes$ kubectl get configmap my-map
NAME     DATA   AGE
my-map   1      95s
cloud_user@ionshipman1c:~/kubernetes$ kubectl get configmap my-map -o yaml
apiVersion: v1
data:
  school: LinuxAcademy
kind: ConfigMap
metadata:
  creationTimestamp: 2018-12-12T14:20:06Z
  name: my-map
  namespace: default
  resourceVersion: "225686"
  selfLink: /api/v1/namespaces/default/configmaps/my-map
  uid: 05dd3a17-fe19-11e8-925b-0ae156f25116
```  

Configmaps are ideal for decoupling the environmental variables from the deployment yamls. 

## Scaling Applications 

1. `kubectl scale deployment/nginx-deployment --replicas=3`
2. Editing the yaml

## Self-Healing Applications 
Constantly checks the status of the cluster and keeps the cluster in the same state.

Replica Controllers (Depracated for deployments) - https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/

# Scheduling (5% of the Exan) 

## Labels and Selectors 

Labels are key value pairs and must be unique 

`kubectl get pods -l app=nginx`  

`kubectl label pod mysql-xxxxxxxx-xxxx test=sure` 
`kubectl describe pod -l test=sure` 

TLDR; Label shit to save time. 

# Excercise: Label all the things 

1. Label each node 
```
cloud_user@ionshipman1c:~$ kubectl get nodes
NAME                           STATUS   ROLES    AGE   VERSION
ionshipman1c.mylabserver.com   Ready    master   32d   v1.12.2
ionshipman2c.mylabserver.com   Ready    <none>   32d   v1.12.2
ionshipman3c.mylabserver.com   Ready    <none>   32d   v1.12.2
cloud_user@ionshipman1c:~$ kubectl label nodes ionshipman1c.mylabserver.com color=black
node/ionshipman1c.mylabserver.com labeled
cloud_user@ionshipman1c:~$ kubectl label nodes ionshipman2c.mylabserver.com color=red
node/ionshipman2c.mylabserver.com labeled
cloud_user@ionshipman1c:~$ kubectl label nodes ionshipman3c.mylabserver.com color=green
node/ionshipman3c.mylabserver.com labeled
cloud_user@ionshipman1c:~$

``` 

2. Label all pods running in a namespace 

```
kubectl label pods -n default running=beforeLabels --all
kubectl get pods -l running=beforeLabels -n default
kubectl label pods --all -n default tier=linuxAcademyCloud
kubectl get pods -l running=afterLabels,tier=linuxAcademyCloud
```

3. Update alpine deployment 
```
apiVersion: v1
kind: Pod
metadata:
  name: alpine
  namespace: default
  labels:
    running: afterLabels
spec:
  containers:
  - name: alpine
    image: alpine
    command:
      - sleep
      - "60"
  restartPolicy: Always
```

## DaemonSets 

Scheduling case for Kubernetes which places, one deployment on each node. 
```
cloud_user@ionshipman1c:~/kubernetes/manifests/linux-academy$ kubectl get daemonsets -n kube-system
NAME                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                     AGE
kube-flannel-ds-amd64     3         3         3       3            3           beta.kubernetes.io/arch=amd64     32d
kube-flannel-ds-arm       0         0         0       0            0           beta.kubernetes.io/arch=arm       32d
kube-flannel-ds-arm64     0         0         0       0            0           beta.kubernetes.io/arch=arm64     32d
kube-flannel-ds-ppc64le   0         0         0       0            0           beta.kubernetes.io/arch=ppc64le   32d
kube-flannel-ds-s390x     0         0         0       0            0           beta.kubernetes.io/arch=s390x     32d
kube-proxy                3         3         3       3            3           <none>                            32d

cloud_user@ionshipman1c:~/kubernetes/manifests/linux-academy$ kubectl describe daemonset kube-flannel-ds-amd64 -n kube-system
Name:           kube-flannel-ds-amd64
Selector:       app=flannel,tier=node
Node-Selector:  beta.kubernetes.io/arch=amd64
Labels:         app=flannel
                tier=node
Annotations:    kubectl.kubernetes.io/last-applied-configuration:
                  {"apiVersion":"extensions/v1beta1","kind":"DaemonSet","metadata":{"annotations":{},"labels":{"app":"flannel","tier":"node"},"name":"kube-f...
Desired Number of Nodes Scheduled: 3
Current Number of Nodes Scheduled: 3
Number of Nodes Scheduled with Up-to-date Pods: 3
Number of Nodes Scheduled with Available Pods: 3
Number of Nodes Misscheduled: 0
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:           app=flannel
                    tier=node
  Service Account:  flannel
  Init Containers:
   install-cni:
    Image:      quay.io/coreos/flannel:v0.10.0-amd64
    Port:       <none>
    Host Port:  <none>
    Command:
      cp
    Args:
      -f
      /etc/kube-flannel/cni-conf.json
      /etc/cni/net.d/10-flannel.conflist
    Environment:  <none>
    Mounts:
      /etc/cni/net.d from cni (rw)
      /etc/kube-flannel/ from flannel-cfg (rw)
  Containers:
   kube-flannel:
    Image:      quay.io/coreos/flannel:v0.10.0-amd64
    Port:       <none>
    Host Port:  <none>
    Command:
      /opt/bin/flanneld
    Args:
      --ip-masq
      --kube-subnet-mgr
    Limits:
      cpu:     100m
      memory:  50Mi
    Requests:
      cpu:     100m
      memory:  50Mi
    Environment:
      POD_NAME:        (v1:metadata.name)
      POD_NAMESPACE:   (v1:metadata.namespace)
    Mounts:
      /etc/kube-flannel/ from flannel-cfg (rw)
      /run from run (rw)
  Volumes:
   run:
    Type:          HostPath (bare host directory volume)
    Path:          /run
    HostPathType:
   cni:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/cni/net.d
    HostPathType:
   flannel-cfg:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      kube-flannel-cfg
    Optional:  false
Events:        <none>
```

* Annonations are for notes, unlike labels which are used for selecting pods. 

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-daemonset
  namespace: kube-system
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: nginx
        image: nginx:1.9.1
        ports:
        - containerPort: 80
```

## Resouce Limits and Pods 

For the exam, how it handles resource limits, taints etc. 

Taints - repel workloads from the node. ex. Have to add a toleration to allow a pod to run on it.  


Adding and removing taints.  

Removing: `kubectl taint nodes ionshipman1c.mylabserver.com node-role.kubernetes.io/master-`
Adding: `kubectl taint node ionshipman1c.mylabserver.com node-role.kubernetes.io=master:NoSchedule` 

Specify memory requests and limits. If the pod exceeds the amount of memory it will be allowed if the node has memory, but eligible for termination. 

** If a Taint and Toleration match, the taint will be ignored and the pod can be scheduled on the node. 

podAffinity can ensure pods end up on the same node together. 
Anti podAffinity can ensure pods DO NOT end up on the same node together.

# Manually Scheduling Pods 

Using labels and node selector 

`kubectl label node ionshipman3c.mylabserver.com network=gigabit` 

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webhead
spec:
  replicas: 1
  selector:
    matchLabels:
      run: webhead
  template:
    metadata:
      labels:
        run: webhead
    spec:
      containers:
      - image: nginx
        name: webhead
        ports:
        - containerPort: 80
          protocol: TCP
      nodeSelector:
        network: gigabit
```
`kubectl describe pods -l run=webhead` 

# Logging and Monitoring (5% of the Exam) 

## Monitoring 

Monitor nodes, containers, pods, service and provide end-users resource usage information. 

Heapster -- Cluster-wide aggregator of monitoring and event data  

Kubelet/cAdvisor on the node -> Heapster runs on single node with kubelet/Cadvisor (communicates with the master) -> Storage backend 

cAdvisor - open source container resouce usage and performance analysis agent. 
  - Auto-discovers all containers on a node and collects CPU, memory, file systems and network usage statistics. 
  - Provides the overall machine usage by analyzing the 'root' container on the machine. 
  - Exposes a simple UI for local containers on port 4194. 

- Kubelet acts as a bridge between the kubernetes master and the nodes. 
- Manages the pods and containers running on a node. 
- Translates each pod into the containers making it up. 
- Obtains usage statistics from cAdvisor. 
- Exposes the aggregated pod resource usage statistics via a REST API.  
- Grafana with InfluxDB
- Heapster is setup to use this storage backend by default. 
- InfluxDB and Grafana run in pods. 
- Pod exposes itself as a Kubernetes service which is how Heapster discovers it. 
- Grafana container servers Grafana's UI whcih provides a dashboard. 

# Managing Logs

## Managing Logs

Logs from pods: `kubectl logs <container_name>` 
Kubernetes system logs: `/var/log/containers`  





























































































