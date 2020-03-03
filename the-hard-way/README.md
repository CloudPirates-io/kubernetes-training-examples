# Kubernetes the hard way

In this guide, we will create a kubernetes single-master cluster with
all basic components installed, to run a basic k8s. Warning: This
guide is only to showcase all k8s components and should never be used
in any case other than testing and demonstrating k8s.


### Prerequisites
We need a single Ubuntu Server with working internet
connection and a valid hostname. Tested with ubuntu 18.04 and 
kubernetes 1.14.2


### Walktrough

Update and Upgrade
```
apt-get update
apt-get upgrade
```


#### Install Docker

```
apt-get install docker.io
```

#### Install vim and curl
```
apt-get install vim curl
```

#### Disable swap
```
swapoff -a
vim /etc/fstab -> Remove swap partition
```

Maybe reboot to completely disable swap.


#### Download and extract required kubernetes binaries

Download Files
`wget https://dl.k8s.io/v1.14.2/kubernetes-server-linux-amd64.tar.gz`

Extract Files
`tar -xzf kubernetes-server-linux-amd64.tar.gz`

Move binaries to /usr/bin/
```
mv kubelet kube-apiserver kubectl kube-controller-manager kube-scheduler kube-proxy /usr/bin/
```


#### Test kubelet

Add manifests directory
```
mkdir -p /etc/kubernetes/manifests
```

Run Kubelet in standalone mode
```
kubelet --pod-manifest-path /etc/kubernetes/manifests &> /etc/kubernetes/kubelet.log &
```

Run first manifest file
```
curl https://gitlab.hd-cms.io/cloudpirate/examples/raw/master/the-hard-way/kubelet-test.yaml > /etc/kubernetes/manifests/kubelet-test.yaml
```

Validate with docker ps / docker logs


#### Install Etcd

```
wget https://github.com/etcd-io/etcd/releases/download/v3.3.13/etcd-v3.3.13-linux-amd64.tar.gz
tar -xzf etcd-v3.3.13-linux-amd64.tar.gz 
cd etcd-v3.3.13-linux-amd64
mv etcd etcdctl /usr/bin/
```

Start and Validate Etcd
```
etcd  --listen-client-urls http://0.0.0.0:2379 --advertise-client-urls http://localhost:2379 &> /etc/kubernetes/etcd.log &
etcdctl cluster-health
```

#### Install API-Server

Start API-Server connected to Etcd. Disabled ServiceAccount to use 
api-server without controller-manger and default serviceaccounts 
created.
```
kube-apiserver --etcd-servers=http://localhost:2379 --service-cluster-ip-range=10.0.0.0/16 --bind-address=0.0.0.0 --insecure-bind-address=0.0.0.0 --disable-admission-plugins=ServiceAccount &> /etc/kubernetes/apiserver.log &
```

Test API-Server `curl http://localhost:8080/api/v1/nodes`

Connect kubectl and add autocompletion
```
apt-get install bash-completion
echo 'source /usr/share/bash-completion/bash_completion' >>~/.bashrc
echo 'source <(kubectl completion bash)' >>~/.bashrc
```
Don´t forget to reload your terminal session.

Currently no connection `kubectl cluster-info` and an
empty config: `kubectl config view`

Add the cluster and the context:
```
# cluster
kubectl config set-cluster kube-from-scratch --server=http://localhost:8080
# context
kubectl config set-context kube-from-scratch --cluster=kube-from-scratch
```

Enable context: `kubectl config use-context kube-from-scratch`

You can also display the config file: `cat .kube/config`


Check connection by getting all resources:
```
kubectl get all --all-namespaces 
```

Kill current kubelet running in standalone mode: `pkill -f kubelet`

### Registering your kubelet to the api-server

Register Kubelet to api-server. Maybe you got some error on first 
start, then simply try again:
```
kubelet --register-node --kubeconfig=".kube/config" &> /etc/kubernetes/kubelet.log &
```

Validate node is registered
```
kubectl get nodes
```

Display, that the manifests are now ignored
```
docker ps
ls /etc/kubernetes/manifests/
```

Create a pod over kubectl
```
# kubectl-test.yaml
curl https://gitlab.hd-cms.io/cloudpirate/examples/raw/master/the-hard-way/kubectl-test.yaml > ~/kubectl-test.yaml

# apply
kubectl apply -f kubectl-test.yaml
```

>Instead of downloading and applying the file, you could simplify to one
>single command: kubectl apply -f https://gitlab.hd-cms.io/cloudpirate/examples/raw/master/the-hard-way/kubectl-test.yaml

Describe pod, no scheduler actions displayed
```
kubectl describe po test-kubectl 
```

#### Install scheduler
Pod are not created… we had no scheduling in place. Add Scheduler:
```
kube-scheduler --master=http://localhost:8080/ &> /etc/kubernetes/scheduler.log &
```

Describe the pod again, now scheduler works, but taint deny scheduling of the pod
```
kubectl describe node
```
Taints: node.kubernetes.io/not-ready:NoSchedule - remove taint:
```
kubectl taint node node01 node.kubernetes.io/not-ready:NoSchedule-
```

pod is creating now: `kubectl get po`

Try to create a deployment

```
# deployment-test.yaml
curl https://gitlab.hd-cms.io/cloudpirate/examples/raw/master/the-hard-way/deployment-test.yaml > ~/deployment-test.yaml
kubectl apply -f deployment-test.yaml
```

No pods are scheduled, because we have no controller manager to manage 
the existing test-deployment.
```
kubectl get deployments
kubectl get po
```
### Install controller-manager

Start Controller Manager
```
kube-controller-manager --master=http://localhost:8080 &> /etc/kubernetes/controller-manager.log &
```

Check deployment
```
kubectl get deployments
kubectl get po
```

Also the controller-manager created the default serviceaccounts. We 
could now start the api-server with service account support, for 
testing simply ignore this

```
kubectl get serviceaccounts
```

Deploy Service
```
# add service-test.yaml
curl https://gitlab.hd-cms.io/cloudpirate/examples/raw/master/the-hard-way/service-test.yaml > ~/service-test.yaml
# apply
kubectl apply -f service-test.yaml
```

Check endpoint created
```
kubectl get service
kubectl describe service test-service
```

Service not reachable: `curl 10.X.XXX.XXX:80`

#### Install kube-proxy
Start kube-proxy
```
kube-proxy --master=http://localhost:8080/ &> /etc/kubernetes/proxy.log &
```
