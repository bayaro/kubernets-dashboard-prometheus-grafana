###  Web UI (Dashboard)

Initial config: [dashboard.yaml](https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml)

As described it allows access only from localhost. But as we are
using vagrant we need to organise a tunnel or we can change default
service spec type from _ClusterIP_ to _NodePort._

```
grep -i node dashboard.yaml
```
>       nodePort: 30000
>   type: NodePort

```
kubectl create -f dashboard.yaml
```
> secret "kubernetes-dashboard-certs" created
> serviceaccount "kubernetes-dashboard" created
> role.rbac.authorization.k8s.io "kubernetes-dashboard-minimal" created
> rolebinding.rbac.authorization.k8s.io "kubernetes-dashboard-minimal" created
> deployment.apps "kubernetes-dashboard" created
> service "kubernetes-dashboard" created

#### Looking for the node port value of the _dashboard_ pod

```
kubectl -n kube-system get service kubernetes-dashboard
```
> NAME                   TYPE     CLUSTER-IP     EXTERNAL-IP PORT(S)                AGE
> kubernetes-dashboard   NodePort 10.107.180.249 <none>      443:_**30000**_/TCP     4m

#### _Dashboard_ GUI requires _Kubeconfig_ or _Token_

Looking for token:

```
kubectl -n kube-system get secret | grep kubernetes-dashboard
```
> kubernetes-dashboard-certs       Opaque                              0   26m
> kubernetes-dashboard-key-holder  Opaque                              2   26m
> kubernetes-dashboard-token-rpjtg kubernetes.io/service-account-token 3   26m

Amazing, but unfortunately all these tokens do not allow us access.

Fortunately the system contains _namespace-controller-token-*_
that we can use :)

```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep namespace-controller-token | awk '{print $1}')
```
> Name:        namespace-controller-token-jfjzj
> Namespace:   kube-system
> Labels:      <none>
> Annotations: kubernetes.io/service-account.name=namespace-controller
> kubernetes.io/service-account.uid=30be9149-580d-11e8-b533-02ee24bb8af6
> Type: kubernetes.io/service-account-token
> Data
> ====
> namespace:  11 bytes
> token:      <long long token will be here>

Voila. Dashboard is working and we can use GUI to observer the
kubernets cluster (currently only with one node).
