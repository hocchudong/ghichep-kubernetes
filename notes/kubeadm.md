# Cài đặt k8s cluster trên ubuntu 16.04 sử dụng công cụ kubeadm

# 1. Giới thiệu
- Kubeadm là một công cụ giúp tự động hóa quá trình cài đặt và triển khai kubernetes trên môi trường Linux, do chính kubernetes hỗ trợ.
- kubeadm hỗ trợ các nền tảng là Ubuntu 16.04, CentOS 7 or HypriotOS v1.0.1+.
- Phiên bản mới nhất của kubeadm là 1.6 dưới dạng beta. Và được kubernetes khuyến cáo là không sử dụng trong môi trường production.

# 2. Mô hình
- Trong bài viết này, tôi sử dụng phiên bản kubeadm 1.6 và chạy trên nền tảng Ubuntu 16.04.
- Cụm cluster có 3 nodes với 1 node master và 2 nodes minion.

![](http://i.imgur.com/qZptWKZ.png)

- master: 172.16.69.237
- minion1: 172.16.69.229
- minion2: 172.16.69.216

# 3. Cài đặt
## 3.1 Cài đặt kubelet and kubeadm trên tất cả các node.
- Bạn phải cài đặt các packages sau đây trên tất cả các node.
  - docker.
  - kubelet.
  - kubectl.
  - kubeadm.

```sh
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
# Install docker if you don't have it already.
apt-get install -y docker-engine
apt-get install -y kubelet kubeadm kubectl kubernetes-cni
```

## 3.2 Khởi tạo master.
- Node master sẽ có nhiệm vụ quản lý các thành phần, các node, bao gồm `etcd` và `API server` để giao tiếp với các node.
- Trên node master, bạn chạy lệnh sau để khởi tạo master.
```sh
kubeadm init
```

- Mặc định thì kubernetes tự động cài đặt với dải mạng gắn với default gateway. Nếu bạn muốn sử dụng interfaces khác, chạy lệnh dưới đây:
```sh
kubeadm init --apiserver-advertise-address <ip-address>
```

- Để xem thêm về các options khác của kubeadm, bạn vào trang dưới đây:
https://kubernetes.io/docs/admin/kubeadm/

- Kết quả
```sh
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run (as a regular user):

  sudo cp /etc/kubernetes/admin.conf $HOME/
  sudo chown $(id -u):$(id -g) $HOME/admin.conf
  export KUBECONFIG=$HOME/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  http://kubernetes.io/docs/admin/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token 32b077.fef34270301cc34a 172.16.69.237:6443
```

- Như thông báo ở trên, các bạn tiếp tục thực hiện các bước sau để chạy được lệnh `kubectl`:
```sh
sudo cp /etc/kubernetes/admin.conf $HOME/
  sudo chown $(id -u):$(id -g) $HOME/admin.conf
  export KUBECONFIG=$HOME/admin.conf
```

- Mặc định thì các pods, services sẽ không chạy trên node master mà chỉ chạy trên các node minion. Nếu muốn chạy trên node master, bạn thực hiện lệnh dưới đây:
```sh
kubectl taint nodes --all node-role.kubernetes.io/master-
```

## 3.3 Cài đặt pod network:
- Chúng ta cần phải cài đặt network trên node **master** thì các pods mới có thể giao tiếp được với nhau.
- Lưu ý, phải cài đặt network trước khi deploy các ứng dụng khác.
- Hiện tại, thì kubernetes hỗ trợ các giải pháp network sau:
  - Calico is a secure L3 networking and network policy provider.
  - Canal unites Flannel and Calico, providing networking and network policy.
  - Cilium is a L3 network and network policy plugin that can enforce HTTP/API/L7 policies transparently. Both routing and overlay/encapsulation mode are supported.
  - Contiv provides configurable networking (native L3 using BGP, overlay using vxlan, classic L2, and Cisco-SDN/ACI) for various use cases and a rich policy framework. Contiv project is fully open sourced. The installer provides both kubeadm and non-kubeadm based installation options.
  - Flannel is an overlay network provider that can be used with Kubernetes.
  - Romana is a Layer 3 networking solution for pod networks that also supports the NetworkPolicy API. Kubeadm add-on installation details available here.
  - Weave Net provides networking and network policy, will carry on working on both sides of a network partition, and does not require an external database

- Trong bài này, tôi sẽ sử dụng **Weave Net** để cài đặt.
```sh
kubectl apply -f https://git.io/weave-kube-1.6
```

- Sau khi cài đặt network, thì pod `kube-dns` đã chạy, ta có thể kiểm tra bằng lệnh sau:
```sh
kubectl get pods --all-namespaces
```

## 3.4 Join các node vào cluster.
- Khi khởi tạo node master, lúc hiện kết quả thành công sẽ có dòng lệnh `kubeadm join --token 32b077.fef34270301cc34a 172.16.69.237:6443`.
- Để join các node vào cluster, ta chạy lên trên ở các node **minion**
```sh
kubeadm join --token 32b077.fef34270301cc34a 172.16.69.237:6443
```

```sh
Node join complete:
* Certificate signing request sent to master and response
  received.
* Kubelet informed of new secure connection details.

Run 'kubectl get nodes' on the master to see this machine join.
```

- Để kiểm tra các node, ta chạy lệnh `kubectl get nodes` trên node master:
```sh
kubectl get nodes
```

```sh
root@master:~# kubectl get nodes
NAME      STATUS    AGE       VERSION
master    Ready     1d        v1.6.1
minion1   Ready     1d        v1.6.1
minion2   Ready     1d        v1.6.1
root@master:~# 
```

Như vậy là quá trình cài đặt cụm cluster kubernetes đã thành công.

## 3.5 Cài đặt dashboard 
- Dashboard cho phép ta xem, chỉnh sửa, xóa các pods, services trên giao diện web.
- Cài đặt, chạy lệnh sau:
```sh
$ kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/kubernetes-dashboard.yaml
```

- Sau đó, chạy lệnh sau để có thể truy cập:
```sh
kubectl proxy --address='0.0.0.0' --accept-hosts='^*$'
```

- Truy cập dashboard ở địa chỉ:
```sh
http://172.16.69.237:8001/ui
```

Trong đó `172.16.69.237` là địa chỉ ip của master node.

# 4. Triển khai ứng dụng nginx trên cluster.
- Trên master, chạy lệnh sau:
```sh
kubectl run nginx --image=nginx --replicas=5 --port=80
```

Trong đó, với option replicas = 5 thì sẽ tạo ra 5 container (5pods) được phân bố đều trên 2 node minion. Và kubernetes sẽ tự động sử dụng thuật toán cân bằng tải round-robin để thực hiện cân bằng tải.

- Expose the deployment to an external IP address: Xuất ra port trên node master để truy cập từ bên ngoài.
```sh
kubectl expose deployment nginx --type=NodePort
```

- Kiểm tra
```sh
root@master:~# kubectl get svc
NAME         CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   10.96.0.1       <none>        443/TCP        1d
nginx     10.110.36.160   <nodes>       80:31844/TCP   4s
root@master:~# 
```

```sh
root@master:~# kubectl describe svc nginx
Name:                   nginx
Namespace:              default
Labels:                 run=nginx
Annotations:            <none>
Selector:               run=nginx
Type:                   ClusterIP
IP:                     10.100.45.223
Port:                   <unset> 80/TCP
NodePort:               <unset> 31844/TCP
Endpoints:              10.36.0.1:80,10.36.0.2:80,10.36.0.3:80 + 2 more...
Session Affinity:       None
Events:           
```

- Kết quả:
```sh
root@master:~# curl http://10.110.36.160
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

```sh
root@master:~# curl http://172.16.69.237:31844
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
- Thử nghiệm tính năng scale:

Tôi tăng lên 10 container đồng thời chạy nginx với lệnh như sau:

```sh
root@master:~# kubectl scale --replicas=10 deployments/nginx
deployment "nginx" scaled
```

```sh
root@master:~# kubectl describe deployments/my-nginx
Name:                   my-nginx
Namespace:              default
CreationTimestamp:      Thu, 20 Apr 2017 14:13:58 +0700
Labels:                 run=my-nginx
Annotations:            deployment.kubernetes.io/revision=1
Selector:               run=my-nginx
Replicas:               10 desired | 10 updated | 10 total | 10 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
Pod Template:
  Labels:       run=my-nginx
  Containers:
   my-nginx:
    Image:              nginx
    Port:               80/TCP
    Environment:        <none>
    Mounts:             <none>
  Volumes:              <none>
Conditions:
  Type          Status  Reason
  ----          ------  ------
  Available     True    MinimumReplicasAvailable
OldReplicaSets: <none>
NewReplicaSet:  my-nginx-858393261 (10/10 replicas created)
Events:
  FirstSeen     LastSeen        Count   From                    SubObjectPath   Type        Reason                   Message
  ---------     --------        -----   ----                    -------------   --------    ------                   -------
  48m           48m             1       deployment-controller                   Normal      ScalingReplicaSet        Scaled down replica set my-nginx-858393261 to 2
  1h            2m              2       deployment-controller                   Normal      ScalingReplicaSet        Scaled up replica set my-nginx-858393261 to 5
  38s           38s             1       deployment-controller                   Normal      ScalingReplicaSet        Scaled up replica set my-nginx-858393261 to 10
root@master:~# 
```

# 5. Một số lưu ý:
- Khi khởi động lại master, bạn phải thực hiện lệnh sau mới có thể chạy được lệnh `kubectl`:
```sh
export KUBECONFIG=$HOME/admin.conf
```

Nếu không sẽ gặp lỗi:
```sh
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

- Nếu truy cập dashboard bị lỗi này:
```sh
Forbidden 403: User "system:serviceaccount:kube-system:default" cannot list pods in the namespace "default". (get pods)
```

Thì fix bằng cách chạy lệnh này:
```sh
kubectl create clusterrolebinding add-on-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:default
```

ref: https://github.com/kubernetes/dashboard/issues/1800

# 6. Tài liệu tham khảo:
- https://kubernetes.io/docs/getting-started-guides/kubeadm/
- https://kubernetes.io/docs/concepts/cluster-administration/addons/
- https://www.weave.works/docs/net/latest/kube-addon/
- https://github.com/kubernetes/dashboard#kubernetes-dashboard