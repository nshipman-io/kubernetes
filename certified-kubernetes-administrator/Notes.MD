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

## Excercise: Viewing the Logs 

`counter.yml` 
```
apiVersion: v1
kind: Pod
metadata:
  name: counter
  labels:
    demo: logger
spec:
  containers:
  - name: count
    image: busybox
    args: [/bin/sh, -c, 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 3; done']
```

1. View the logs
`kubectl logs counter` 
2. Allow the container to run for a few minutes while viewing the log interactively. 
`kubectl logs -f counter` 
3. Have the command only print the last 10 lines of the log.
`kubectl logs --tail=10 counter` 
4. Look at the logs for the scheduler. Are there any problems? 
`vim /var/log/containers/kube-scheduler-XXXX` 
```
{"log":"E1219 14:50:49.648787       1 reflector.go:134] k8s.io/client-go/informers/factory.go:131: Failed to list *v1.PersistentVolume: Get https://172.31.44.139:6443/api/v1/persistentvolumes?limit=500\u0026resourceVersion=0: net/http: TLS handshake timeout\n","stream":"stderr","time":"2018-12-19T14:50:49.649036579Z"}
```
5. Kubernetes uses etcd for it's key-value store. Examine the logs: 
`vim /var/log/containers/etcd-ionshipman1c.mylabserver.com_kube-system_etcd-XXXX` 
```
{"log":"2019-01-02 14:17:22.005281 I | embed: ClientTLS: cert = /etc/kubernetes/pki/etcd/server.crt, key = /etc/kubernetes/pki/etcd/server.key, ca = , trusted-ca = /etc/kubernetes/pki/etcd/ca.crt, client-cert-auth = true\n","stream":"stderr","time":"2019-01-02T14:17:22.005598809Z"}
{"log":"2019-01-02 14:17:22.008360 I | etcdserver: a874c87fd42044f as single-node; fast-forwarding 9 ticks (election ticks 10)\n","stream":"stderr","time":"2019-01-02T14:17:22.008506513Z"}
{"log":"2019-01-02 14:17:22.590041 I | raft: a874c87fd42044f is starting a new election at term 11\n","stream":"stderr","time":"2019-01-02T14:17:22.590346827Z"}
{"log":"2019-01-02 14:17:22.590075 I | raft: a874c87fd42044f became candidate at term 12\n","stream":"stderr","time":"2019-01-02T14:17:22.590367349Z"}
{"log":"2019-01-02 14:17:22.590093 I | raft: a874c87fd42044f received MsgVoteResp from a874c87fd42044f at term 12\n","stream":"stderr","time":"2019-01-02T14:17:22.590371737Z"}
{"log":"2019-01-02 14:17:22.590121 I | raft: a874c87fd42044f became leader at term 12\n","stream":"stderr","time":"2019-01-02T14:17:22.590375347Z"}
{"log":"2019-01-02 14:17:22.590128 I | raft: raft.node: a874c87fd42044f elected leader a874c87fd42044f at term 12\n","stream":"stderr","time":"2019-01-02T14:17:22.590378892Z"}
```
6. Kubernetes API server runs as a pod in the cluster. Examine the logs.
`vim /var/log/containers/kube-apiserver-ionshipman1c.mylabserver.com_kube-system_kube-apiserver-XXXX` 
```
{"log":"[restful] 2019/01/02 14:17:26 log.go:33: [restful/swagger] listing is available at https://172.31.44.139:6443/swaggerapi\n","stream":"stderr","time":"2019-01-02T14:17:26.79982418Z"}
{"log":"[restful] 2019/01/02 14:17:26 log.go:33: [restful/swagger] https://172.31.44.139:6443/swaggerui/ is mapped to folder /swagger-ui/\n","stream":"stderr","time":"2019-01-02T14:17:26.79984574Z"}
{"log":"[restful] 2019/01/02 14:17:28 log.go:33: [restful/swagger] listing is available at https://172.31.44.139:6443/swaggerapi\n","stream":"stderr","time":"2019-01-02T14:17:28.627682824Z"}
{"log":"[restful] 2019/01/02 14:17:28 log.go:33: [restful/swagger] https://172.31.44.139:6443/swaggerui/ is mapped to folder /swagger-ui/\n","stream":"stderr","time":"2019-01-02T14:17:28.62770902Z"}
{"log":"I0102 14:17:28.683641       1 plugins.go:158] Loaded 8 mutating admission controller(s) successfully in the following order: NamespaceLifecycle,LimitRanger,ServiceAccount,NodeRestriction,Priority,DefaultTolerationSeconds,DefaultStorageClass,MutatingAdmissionWebhook.\n","stream":"stderr","time":"2019-01-02T14:17:28.683914931Z"}
{"log":"I0102 14:17:28.683661       1 plugins.go:161] Loaded 6 validating admission controller(s) successfully in the following order: LimitRanger,ServiceAccount,Priority,PersistentVolumeClaimResize,ValidatingAdmissionWebhook,ResourceQuota.\n","stream":"stderr","time":"2019-01-02T14:17:28.683934878Z"}
{"log":"I0102 14:17:33.246623       1 secure_serving.go:116] Serving securely on [::]:6443\n","stream":"stderr","time":"2019-01-02T14:17:33.25175149Z"}
{"log":"I0102 14:17:33.246681       1 autoregister_controller.go:136] Starting autoregister controller\n","stream":"stderr","time":"2019-01-02T14:17:33.251775853Z"}
{"log":"I0102 14:17:33.246687       1 cache.go:32] Waiting for caches to sync for autoregister controller\n","stream":"stderr","time":"2019-01-02T14:17:33.25178029Z"}
{"log":"I0102 14:17:33.249050       1 available_controller.go:278] Starting AvailableConditionController\n","stream":"stderr","time":"2019-01-02T14:17:33.25178386Z"}
{"log":"I0102 14:17:33.249059       1 cache.go:32] Waiting for caches to sync for AvailableConditionController controller\n","stream":"stderr","time":"2019-01-02T14:17:33.251787227Z"}
{"log":"I0102 14:17:33.249075       1 controller.go:84] Starting OpenAPI AggregationController\n","stream":"stderr","time":"2019-01-02T14:17:33.251790698Z"}
{"log":"I0102 14:17:33.249514       1 crd_finalizer.go:242] Starting CRDFinalizer\n","stream":"stderr","time":"2019-01-02T14:17:33.251793884Z"}
{"log":"I0102 14:17:33.250454       1 apiservice_controller.go:90] Starting APIServiceRegistrationController\n","stream":"stderr","time":"2019-01-02T14:17:33.251796945Z"}
```
or 

`kubectl logs kube-apiserver-ionshipman1c.mylabserver.com -n kube-system`  
```
I0102 14:17:33.441412       1 log.go:172] http: TLS handshake error from 172.31.44.139:34580: EOF
I0102 14:17:33.450331       1 log.go:172] http: TLS handshake error from 172.31.44.139:34584: EOF
I0102 14:17:33.451734       1 log.go:172] http: TLS handshake error from 172.31.47.1:43182: EOF
I0102 14:17:33.457912       1 log.go:172] http: TLS handshake error from 172.31.47.56:44862: EOF
I0102 14:17:33.468958       1 log.go:172] http: TLS handshake error from 172.31.44.139:34622: EOF
I0102 14:17:33.536832       1 cache.go:39] Caches are synced for autoregister controller
I0102 14:17:33.537139       1 cache.go:39] Caches are synced for AvailableConditionController controller
I0102 14:17:33.537455       1 cache.go:39] Caches are synced for APIServiceRegistrationController controller
I0102 14:17:33.553396       1 controller_utils.go:1034] Caches are synced for crd-autoregister controller
I0102 14:17:34.264633       1 storage_scheduling.go:100] all system priority classes are created successfully or already exist.
I0102 14:17:51.397609       1 controller.go:608] quota admission added evaluator for: { endpoints}
E0102 14:31:38.221252       1 watcher.go:208] watch chan error: etcdserver: mvcc: required revision has been compacted
E0102 14:39:18.232895       1 watcher.go:208] watch chan error: etcdserver: mvcc: required revision has been compacted
E0102 14:54:34.267870       1 watcher.go:208] watch chan error: etcdserver: mvcc: required revision has been compacted
I0102 15:01:38.545934       1 trace.go:76] Trace[782359243]: "Get /api/v1/namespaces/default/pods/counter/log" (started: 2019-01-02 15:00:36.29566597 +0000 UTC m=+2597.809146312) (total time: 1m2.250217095s):
Trace[782359243]: [1m2.250217095s] [1m2.192868446s] END
E0102 15:02:46.298266       1 watcher.go:208] watch chan error: etcdserver: mvcc: required revision has been compacted
```

View logs from a dead or evicted pod. 
`kubectl logs --previous <pod_name>`

# Cluster Maintenance (11% of the Exam) 

## Upgrading Kubernetes Components
1. Update the control plane
`apt-get upgrade kubeadm`
Verify: `kubeadm version` 
Plan Upgrade: `sudo kubeadm upgrade plan`
Apply Upgrade: `kubeadm upgrade apply v1.13.1`  

2. Verify CNI version (Flannel in this case)
`kubectl get daemonsets -n kube-system kube-flannel-ds-amd64 -o yaml | grep image:` 
```
image: quay.io/coreos/flannel:v0.10.0-amd64
image: quay.io/coreos/flannel:v0.10.0-amd64
```
3. Upgrade nodes 
drain the nodes: `kubectl drain ionshipman2c.mylabserver.com --ignore-daemonsets`
Upgrade kubelet: `apt update && apt upgrade kubelet`
uncordon the node (allows scheduling): `kubectl uncordon ionshipman2c.mylabserver.com`

## Upgrading Underlying Operating System(s)

drain the node
`kubeadm token list` 
Join a new node to cluster: 
`kubeadm token generate` 
`kubeadmin token create <Token> --ttl 3h` ** ttl = Time To Live 
Copy the join command and run on the new node. 

## Excercise: Mainenance on NOde 
`kubectl drain ionshipman3c.mylabserver.com --ignore-daemonsets` 
`lots-of-nothing.yml`
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lots-of-nothing
  labels:
    app: lots-of-nothing
spec:
  replicas: 6
  selector:
    matchLabels:
      app: lots-of-nothing
  template:
    metadata:
      labels:
        app: lots-of-nothing
    spec:
      containers:
      - name: lots-of-nothing
        image: k8s.gcr.io/pause:2.0
```
`kubectl uncordon ionshipman3c.mylabserver.com`
# Networking (11% of the Exam) 
## Node Networking Configuration 
Inbound Node Port Requirements 

Master Node(s): 
- TCP 6443 - Kubernetes API Server
- TCP 2379-2380 - etcd server client API
- TCP 10250 - Kubelet API
- TCP 10251 - kube-scheduler
- TCP 10252 - kube-controller-manager
- TCP 10255 - Read-only Kubelet API 

Worker Nodes 
- TCP 10250 - Kubelet API 
- TCP 10255 - Read-only Kubelet API
- TCP 30000-32767 - Node Port Services 

## Service Networking 

Nodeport = Every single node in the cluster, assign a port that a load balancer can point to in order to deliver contents of the pod.  
```
cloud_user@ionshipman1c:~/kubernetes/manifests/linux-academy$ kubectl expose deployment webhead --type="NodePort" --port 80
service/webhead exposed
cloud_user@ionshipman1c:~/kubernetes/manifests/linux-academy$ kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        49d
webhead      NodePort    10.99.205.104   <none>        80:31861/TCP   8s
cloud_user@ionshipman1c:~/kubernetes/manifests/linux-academy$ curl localhost:31861
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Every node is running kube-proxy and redirecting requests to the particular port and redirecting it to the pod. 
Cluster IP can redirect requests to a particulr IP address if there is a load balancer. 

## Ingress 

Ingress is an API object that manages external access to the service in a cluster, usually HTTP. 
It can provide load balancing, SSL termination and name-based virtual hosting.  

This could be a gateway managed by a cloud provider or a physical piece of hardware.  
Our cluster network is a set of links, either logical or physical, that facilitate communication within a cluster according to the kubernetes networking model. Examples include overlays such as flannel  
or SDNs such as OpenVSwitch. 

Service is a kubernetes service that identifies a set of pod using label selectors. Services are assumed to have virtual IPs only routable within the cluster network. 

Ingress is a collection of rules that allow inbound conenctions. Can be configured to give services externally-reachable URLs, load balance traffice, terminate SSL, offer name-based virtual hosting. 
User request ingress by POSTing the ingress resource to the API server. An Ingress controller is responsible for fufilling the Ingress, usually by way of a load balancer, though it may also configure the edge router or additional  front hends to help handle to help handle the traffic in a HA manner.  

Not available before Kubernetes 1.1 

Need an ingress controller to satisfy ingress objects. Deploy it on the master, which must be annotated with the approprate class. 

Ingress object contains all the specs needed to configure a load balancer, or proxy server and http rules. Host and path must match the content before the load balancer forwards to the backend

You must manually configure the ingress controller. 

Default backend can route things to a 404 page when the requests doesn't match any hosts and services.

Securing an Ingress: 

Specify secret 
  - TLS private key 
  - Certificate 

Port 443 (Assumes TLS termination)

Multiple hosts are multiplexed on the same port by hostnames specified through the SNI TLS extension. 
The TLS secret must contain keys named tls.crt and tls.key that conain the certificate and private key to use for TLS 

An ingress controller is bootstrapped with a load balancing policy that it applies to al all ingress objects. (e.g. load balancing algorithm, backend weight scheme, etc)
Persistent sessions and dynamic weights not yet exposed
Health checks are not exposed directly though the ingress, readiness probes allow for similar functionality. 

`kubectl get ingress` to return ingress. 
`kubectl edit ingress <ingress_name>` edit ingress using vim.
`kubectl replace -f <file_name>` can update the running Kubernetes object. 

Other ways to expose services without ingress 
- Service. Type=LoadBalancer 
- Service. Type=NodePort 
- Use a port proxy 

## Deploying a load balancer 
```
--- 
apiVersion: v1 
kind: Service 
metadata:  
  name: la-lb-service 
spec: 
  selector: 
    app: la-lb 
  ports: 
  - protocol: TCP 
    port: 80 
    targetPort: 9376 
  clusterIP: 10.0.171.223 
  loadBalancerIP: 78.12.23.17 
  type: loadBalancer
``` 
Need to know the type of the service and ingress rules for diverting outside traffic into the cluster. 

## Configure and Use Cluster DNS 

Excercise: 

Run the `bit-of-nothing` deployment, run `busybox` pod and verify DNS resolution for the `bit-of-nothing` deployment. 
1. `kubectl exec -it busybox -- nslookup bit-of-nothing` 
```
cloud_user@ionshipman1c:~/kubernetes/manifests/linux-academy$ kubectl exec -it busybox -- nslookup bit-of-nothing
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

nslookup: can't resolve 'bit-of-nothing'
command terminated with exit code 1
```

2. Expose the bit of nothing deployment as a ClusterIP service.  
`kubectl expose deployment bit-of-nothing --type=ClusterIP --port 80`

3. Verify that `bit-of-nothing` is now being resolved to an IP address in the cluster 
`kubectl get services`
```
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
bit-of-nothing   ClusterIP   10.104.189.74   <none>        80/TCP         10s
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP        49d
webhead          NodePort    10.99.205.104   <none>        80:31861/TCP   132m
```
`kubectl exec -it busybox -- nslookup bit-of-nothing`
```
cloud_user@ionshipman1c:~/kubernetes/manifests/linux-academy$ kubectl exec -it busybox -- nslookup bit-of-nothing
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      bit-of-nothing
Address 1: 10.104.189.74 bit-of-nothing.default.svc.cluster.local
```
## Container Network Interface (CNI) 

All pods can communicate with all other pods. Each pod has its own IP address, No need for mapping container ports and Backward compatible with VMs: 
  - Port allication 
  - Naming 
  - Service Discovery 
  - Load balancing 
  - Application Configuration 
  - Migration 

Dynamic Port Allocation Problems: 
  - Every application must be configured to know which ports, etc. 
  - API services must inject dynamic port numbers into containers 
  - Services must be able to find one another or be configured to do so . 

Kubernetes Way: 
  - All containers can communicate with each other without NAT 
  - All nodescan communciate with all containers and vice versa without NAT 
  - The IP of a container is the same regardless of which container views it
  - k8s applies IP addresses at the pod level 
  - IP-per-pod -- containers in a pod share a single IP address, like processes in a VM

CNI -- Must be implemented as an executable invoked by the container management system (kubernetes in this case) 
plugin is responsible for 
  - Inserting the network interface into the container network namespace 
  - making necessary changes to the host 
  - assign IP address to the interface 
  - Set up routes consistent with IP address management 

Kubelet: 
  - Default network plugin 
  - Default cluser-wide network 
  - Probes for network plugins on startup 

Flannel - Simple and easy Layer 3 network frabric. flanneld runs on each host (via a DaemonSet) 
  - Allocates subnet least to each host 
  - Stores network configuration, allocated subnets, other data 
  - Packets forwarded using VxLANs 
  - Does not handle network policies 

Calico - Simple networking model, Policy driven network security. Can be run in policy enforement mode (Pair it with Flannel (Canal)) 

# Storage (7% of the Exam) 

## Persistent Volumes 

Native Pod Storage is Ephermeral 

What happens when a container crashes: 
  - Kubelet restarts it (possibly on another node) 
  - File system is re-created from image 
  - Ephemeral files are gone 

Docker Volumes 
  - Directory to disk 
  - Possibly in another container
  - New Volume Drivers

Kubernetes Volumes 
  - Same lifetime as its pod 
  - Data preserved across container restarts 
  - Pod goes away -> Volume goes away 
  - Directory with data 
  - Accessible to containers in a pod 
  - Implementation details determined by volume types  

Using Volumes 
  - Pod spec indicates which volumes is provided for the pod (spec,volumes)
  - Pod spec indicates where to mount these volumes in continers (spec,containers,volumeMounts)
  - Seen from the containers perpective as the file system 
  - Volumes cannot mount to other volumes 
  - no hard links to other volumes 
  - Each pod must specify where each pod must specify where each volume is mounted 

Empty Dir for caching 

`aws ec2 create-volume --availability-zone=us-east-1a --size=10 --volume-type=gp2`

```
apiVersion: v1 
Kind: Pod 
metadata: 
  name: test-pd 
specs: 
  containers:  
  - image: k8s.gcr.io/test-webserver 
    name: test-container 
    volumeMounts: 
    - mountPath: /test-pd 
    name: test-volume 
  volumes: 
  - name: test-volume
  # EBS volume must be created in AWS First
    awsElasticBlockStore: 
      volumeID: <volume-id>
      fsType: ext4
``` 

## Volumes and Their Access Modes 

 PersistentVolume -- API for users that abstracts implementation details of storage 
  - Provisioned storage in the cluster 
  - Cluster resource 
  - Volume plugins have independent lifecycle from pods 
  - Volumes share the lifecycle of the pod; PersistentVolumes do not 
  - API object (YAML) details the implementation 

 Persistent VolumeClaim -- Method for users to claim durable storae regardless of implementation details 
  - Request for storage 
  - Pods consume node resources; PVCs consume PV resources. 
  - Pods can request specific CPU and memory. PVCs can request specific size and access modes.

PVs and PVCs 
- Users and applications do not share identical requirements 
- Administrators should offer a variety of PVs without users worrying about the implementation details. (storageclasses)
- PVs are cluster resources. PVCs are requests for the cluster resource 
- PVCs also act as a "claim check" on a resource. 
- PVs and PVCs have a set lifecycle 
  - Provision
  - Bind 
  - Reclaim 

Provisioning 

- Static 
  - Creates PVs 
  - In the k8S API and available for consumption 

- Dynamic 
  - Used when none of the static PVs match the PVC 
  - Based on StorageClasses 
  - PVC must request a created and configured storage class 
  - Claims requesting nameless class disable dynamic provisioning 

To enable dynamic storage provisoning, DefaultStorageClass admission controller of the API server must be enabled. 

Binding 

- USer creates PVC 
- MAster watches for new PVCs and matches them to PVs 
- Master binds PVC to PV 
- Volume may be more than the request 
- Bind are exclusive 
- PVC -> PV mapping is always 1:1 
- Claims not matched will remained unbound 
- Pods treat PVCs as volumes 
- Cluster checks claim, mounts appropriate volume to pod

Persistent Volume Claim Protection 

- Ensures PVC actively in use do not get removed from the system 
- Ensures PVCs actively in use do not get removed from the system
- PVC considered active when: 
  - The pod status is pending and the pod is assigned to a node
  - The pod status is running 
- If a user deletes a PVC in use, removal is postponed until PVC is not use by any pod 

```
status: Terminating 
Finalizers: [kubernetes.io/pvc-protection]
```

Reclaiming 

- User can delete PVC objects 
- Reclaim policy for a PV tells the cluster what to do with the volume 
- Policies 
  - Retain 
  - Recycle 
  - Delete 
- Policies allow for manual reclamation of a resource 
  - PVC deleted 
  - PV still exists; volume is released  
  - Not yet available because the data is still present 
  - Admin can manually reclaim the volume 

- Administators can configure custom recycler pod template 
- Must contain a volumes specification 
- Volume plugs that support Delete reclaim policy 
  - Removes the Persistent/Volume object from K8s 
  - Associated storage assets in the infrastructure 
  - Dynaically provisioned volumes inheritt reclaim policy of their StorageClass 
- Administrator should configure StorageClass accourding to expectations  

Capcacity 

- PVs have a speciic storage capacity
- Set using "capacity" attribute 
- Storage size is the only resource that can be set or requested 
- future attribute plans: 
  - IOPS 
  - Throughput 
  - ??? 

- Can specify VolumeMode 
  - Raw Block Devices -- "Block" 
  - File systems -- "Filesystem" (default) 

- Must be supported by storage resource provider 
- ReadWriteOnce -- Can be mounted as read/write by one nod eonly (RWO)
- ReadOnlyMany -- Can be mounted read-only by many nodes (ROX) 
- ReadWriteMany -- Can be mounted read/write by many nodes (RWX) 

A volume can onlyu be mounted using one access mode at a time, regardless of the modes that are supported. 

## Applications and Persistent Storage 
```
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: lapv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /var/nfs/general
    server: 172.31.39.246
    readOnly: false
```

`kubectl get pv`
`kubectl get pvc`

Persistent Volume is provisioned by the administrator. 
Persistent Volume Claim is requested by the application or the user. 
# Security (12% of the Exam)
# Troubleshooting (10% of the Exam)




















































































