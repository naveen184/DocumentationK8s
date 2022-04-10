# Kubernetes using kubeadm
In this tutorial we are going to create kubernetes cluster using kubeadm at local instance.
### Kubernetes Architect
* 1 Master Node
* 2 Worker Nodes

## Step 1: Init servers for k8s components

For initializing servers we will use:

* VirtualBox
* Vargant
* [github repo]

To install vargant follow this [link](https://www.vagrantup.com/downloads)

To install virtualBox follow this [link](https://www.virtualbox.org/wiki/Downloads)

Don't spin vm directly into virtualbox. Use vagrant file to spin vms on virtualbox.

Clone [github repo] into a local directory and check for vargrant file

Check vagrant status inside the clone directory as it has a .vagrant dir by:
``` 
vagrant status 
```

It will show 
```
kubemaster                not created (virtualbox)
kubenode01                not created (virtualbox)
kubenode02                not created (virtualbox)
```
To create the above instance run
```
vagrant up
```
This command will provision all the three vms on virtual box

Check ssh conectivity using below command
```
vagrant ssh kubemaster
vagrant ssh kubenode01
vagrant ssh kubenode02
```
To logout form the instance we use 
```
logout
```

## Step 2: Install Container Run Time Engine
We are going to use Docker for the container run time engine.

Install docker on all the nodes.

For Installation of Docker you can follow this [link](https://docs.docker.com/engine/install/ubuntu/) too
* Update the apt package index and install packages to allow apt to use a repository over HTTPS:
```
$ sudo apt-get update
$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
* Add Docker’s official GPG key:
```
$  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
* Use the following command to set up the stable repository.
```
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
* Install Docker Engine
Update the apt package index, and install the latest version of Docker Engine and containerd, or go to the next step to install a specific version:
```
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```
```
$ sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
```

## Step 3: Enable Bridge Network
To check the  bridge network
```
lsmod | grep br_netfilter
```
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```
## Step 4: Install kubeadm, kubectl and kubelet
Install kubeadm, kubectl and kubelet on all nodes:
```
$ sudo apt-get update
$ sudo apt-get install -y apt-transport-https ca-certificates curl
$ sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
$ echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
```
Check kubeadm version and apply particular version. Also version for all the component must be same
```
$ sudo apt-get install -y kubelet=1.21.0-00 kubeadm=1.21.0-00 kubectl=1.21.0-00
$ sudo apt-mark hold kubelet kubeadm kubectl
```
## Step 5: Bootstrap a kubernetes cluster
Now bootstrap a kubernetes cluster using kubeadm.
Before this we need to get the master node ip address, for this we will run ifconfig and check eth0 network
```
$ ifconfig
```
now we will use this ip address under apiserver-advertise-address in the command
```
kubeadm init --apiserver-cert-extra-sans=controlplane --apiserver-advertise-address 10.2.223.3 --pod-network-cidr=10.244.0.0/16
```
As the command completes it will provide cluster credentails and token for adding nodes. copy them and keep safe

## Step 6: Setup Kubeconfig for the cluster access
Kubeadm will save the credential under /etc/kubernetes/admin.conf we ned to copy them into .kube/config file
```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Step 7: Adding Node to Cluster
Switch to the node server and run the below gather command from the master cluster
```
kubeadm join 10.17.68.12:6443 --token aaekvx.pb6fzkirbhqkxxan --discovery-token-ca-cert-hash sha256:65dccb7472414b8e1e4640c495c01896b57478fa2a744e28f662b71a6952783d

```
Now jump to masterplane and run
```
kubectl get nodes
```

## Step 8: Adding cluster network interface CNI
Here we will install flannel. You can install waeve or something else as you prefer
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```












[github repo]: <https://github.com/devopshimanshu/certified-kubernetes-administrator-course>