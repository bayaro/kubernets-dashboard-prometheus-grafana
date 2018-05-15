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

####  First attempt to run kubelet
```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
sudo systemctl status kubelet
```
> ● kubelet.service - kubelet: The Kubernetes Node Agent
    Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
    Active: activating (auto-restart) (Result: exit-code) since Tue 2018-05-15 06:35:38 UTC; 3s ago
      Docs: http://kubernetes.io/docs/
   Process: 5871
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS
$KUBELET_SYSTEM_PODS_ARGS $KUBELET_NETWORK_ARGS $KUBELET_DNS_ARGS
$KUBELET_AUTHZ_
  Main PID: 5871 (code=exited, status=255)
May 15 06:35:38 ubuntu-xenial systemd[1]: kubelet.service: Unit entered failed state.
May 15 06:35:38 ubuntu-xenial systemd[1]: kubelet.service: Failed with result 'exit-code'.

Ooops :) kubelet does not configured at all at this point.

#### Using kubeadm to Create a Cluster

As we need a two nodes cluster so we need to use a public interface
for vagrant machines (option --_apiserver-advertise-address_).

For using the Flannel pod network add-on we need --_pod-network-cidr_
option. It can be any private subnet.

```
sudo kubeadm init --apiserver-advertise-address=<public master mode ip> --pod-network-cidr=10.244.0.0/24
```
> …
Your Kubernetes master has initialized successfully!
…

Run below commands to allow usage of kubectl by non-privileged user

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Save in the appropriate place the suggested command:

```
kubeadm join <public master mode ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<sha256hash>
```

This command should be execute on every additional node to join it to the cluster.

Check pods in the cluster

```
kubectl get pods --all-namespaces
```
>NAMESPACE   NAME                                  READY STATUS  RESTARTS   AGE
kube-system  etcd-ubuntu-xenial                    1/1   Running 0          2m
kube-system  kube-apiserver-ubuntu-xenial          1/1   Running 0          2m
kube-system  kube-controller-manager-ubuntu-xenial 1/1   Running 0          2m
kube-system  kube-dns-86f4d74b45-r7n7r             3/3   Running 0          2m
kube-system  kube-proxy-pq278                      1/1   Running 0          2m
kube-system  kube-scheduler-ubuntu-xenial          1/1   Running 0          2m

#### Releasing a taintment of master node

As we work yet with only one node it will be better to untaint a master node.

```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

#### Installing a pod network (Flannel)

config: [flannel.yml](https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml)

```
kubectl create -f conf/flannel.yaml
```
> clusterrole.rbac.authorization.k8s.io "flannel" created
clusterrolebinding.rbac.authorization.k8s.io "flannel" created
serviceaccount "flannel" created
configmap "kube-flannel-cfg" created
daemonset.extensions "kube-flannel-ds" created

```
kubectl get pods --all-namespaces | grep flannel
```
> kube-system   kube-flannel-ds-zkcjb   1/1       Running   0          57s

#### Other nodes

NB!!! very important: all
below values should be unique for every host in a cluster!!!!

1) MAC address of the **all** network interfaces across all nodes. Check with **ifconfig -a**
2) product_uuid: Check with **sudo cat /sys/class/dmi/id/product_uuid**
3) hostname: Check with **hostname**

Installing kubeadm, kubelet and kubectl

```
sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF > kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo mv kubernetes.list /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y docker.io kubelet kubeadm kubectl
sudo apt list docker.io kubelet kubeadm kubectl
```

Join the node to the cluster

```
kubeadm join <public master mode ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<sha256hash>
```
> node "ubuntu-xenial" untainted

**Sources:**
1. [Installing kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/)
2. [Using kubeadm to Create a Cluster](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)
