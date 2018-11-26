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

### Raw Kubernetes Install -- Ubuntu 

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
1. Install docker 
    - `apt install docker.io` 
    - enable docker
2. Install kubernetes 
```
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add 
  sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
  sudo apt install kubeadm
```
    