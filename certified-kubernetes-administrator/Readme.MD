# Linux Academy - Certified Kubernetes Administrators 


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
- `kubectl run nginx --image=nginx` or `kubectl create secret genetic hello-kube-conf --from-file=hello-kube.conf`

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