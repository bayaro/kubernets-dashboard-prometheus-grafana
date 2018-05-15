###  Kubernetes deployment

#### Initial requirements

VirtualBox

Vagrant

#### Working Vagrant OS

I have started with Debian 9.
> Package 'docker.io' has no installation candidate

Ubuntu 16.04 was chosen.

#### Install Docker from Ubuntu’s repositories

```
sudo apt-get update 
sudo apt-get install -y docker.io 
sudo apt list docker* 
```
> docker.io/xenial-updates,now 1.13.1-0ubuntu1~16.04.2 amd64 [installed]

#### Installing kubeadm, kubelet and kubectl
```
sudo apt-get install -y apt-transport-https curl 
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - 
cat <<EOF > kubernetes.list  
deb	http://apt.kubernetes.io/ kubernetes-xenial main  
EOF 
sudo mv kubernetes.list /etc/apt/sources.list.d/kubernetes.list 
sudo apt-get update 
sudo apt-get install -y kubelet kubeadm kubectl 
sudo apt list kubelet kubeadm kubectl
```
> kubeadm/kubernetes-xenial,now 1.10.2-00 amd64 [installed]  
> kubectl/kubernetes-xenial,now 1.10.2-00 amd64 [installed]  
> kubelet/kubernetes-xenial,now 1.10.2-00 amd64 [installed]

#### Cgroup driver check
```sudo docker info | grep -i cgroup ```
> Cgroup Driver: cgroupfs

```sudo grep -i cgroup /etc/systemd/system/kubelet.service.d/10-kubeadm.conf```
> Environment="KUBELET_--cgroup-driver=cgroupfsNETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
