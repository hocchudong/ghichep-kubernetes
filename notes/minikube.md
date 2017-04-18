# Minikube
- Minikube là công cụ giúp bạn dễ dàng chạy kubernetes ngay trên máy tính của bạn.
- Các tính năng mà minikube cung cấp đó là:
  - DNS
  - NodePorts
  - ConfigMaps and Secrets
  - Dashboards
  - Container Runtime: Docker, and rkt
  - Enabling CNI (Container Network Interface)
  - Ingress

# 1. Cài đặt
Trong bài hướng dẫn này, tôi chạy minikube trên môi trường ubuntu 14.04.

# 1.1 Cài đặt các ứng dụng yêu cầu:
- Máy tính phải hỗ trợ tính năng VT-x/AMD-v virtualization trong BIOS
```sh
root@adk:/home/adk# cat /proc/cpuinfo | grep 'vmx\|svm'
```
![](http://storage7.static.itmages.com/i/17/0418/h_1492498532_7579004_5b465fdce1.png)

- Cài đặt VirtualBox hoặc KVM. Ở đây, tôi sẽ sử dụng VirtualBox.

```sh
wget http://download.virtualbox.org/virtualbox/5.1.18/virtualbox-5.1_5.1.18-114002~Ubuntu~trusty_amd64.deb

dpkg -i virtualbox-5.1_5.1.18-114002~Ubuntu~trusty_amd64.deb
```

- Cài đặt `kubectl`:
```sh
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
``` 

# 1.2 Cài đặt Minikube
```sh
curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.18.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```
# 2. Deploy
- Start local Kubernetes cluster:
```sh
root@adk:/home/adk# minikube start
Starting local Kubernetes cluster...
Starting VM...
SSH-ing files into VM...
Setting up certs...
Starting cluster components...
Connecting to cluster...
Setting up kubeconfig...
Kubectl is now configured to use the cluster.
```
Sau khi chạy lệnh này, môi trường minikube đã được setup xong và các bạn có thể khai báo các pods, services để chạy. 

- Kiểm tra các node có trong cluster
```sh
root@adk:/home/adk# kubectl get nodes
NAME       STATUS    AGE       VERSION
minikube   Ready     7d        v1.6.0
root@adk:/home/adk# 
```

- Để kiểm tra các pods đang chạy, bạn dùng lệnh
```sh
root@adk:/home/adk# kubectl get pods --all-namespaces
NAMESPACE     NAME                          READY     STATUS    RESTARTS   AGE
kube-system   kube-addon-manager-minikube   1/1       Running   6          7d
kube-system   kube-dns-v20-xd09t            3/3       Running   15         7d
kube-system   kubernetes-dashboard-1d83q    1/1       Running   5          7d
root@adk:/home/adk# 
```

Mặc định thì kubernetes tạo và khởi chạy các pods này.

- Xem các services đang chạy
```sh
root@adk:/home/adk# kubectl get svc --all-namespaces
NAMESPACE     NAME                   CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
default       kubernetes             10.0.0.1     <none>        443/TCP         7m
kube-system   kube-dns               10.0.0.10    <none>        53/UDP,53/TCP   7d
kube-system   kubernetes-dashboard   10.0.0.77    <nodes>       80:30000/TCP    7d
root@adk:/home/adk# 
```
- Minikube cung cấp dashboard cho phép ta xem, tạo, chỉnh sửa các pods, services. Để khởi chạy dashboard, các bạn dùng lệnh:
```sh
minikube dashboard
```
Sau đó, truy cập vào địa chỉ http://ip:30000 để truy cập vào dashboard. Trong đó, địa chỉ ip là giá trị
```sh
minikube ip
```

![](http://storage9.static.itmages.com/i/17/0418/s_1492500264_5772794_7af103cdf2.png)

- Stop minikube
```sh
$ minikube stop
Stopping local Kubernetes cluster...
Machine stopped.
```
# 3. Demo deploy ứng dụng nginx

- Tạo `deployment` có tên là nginx từ image nginx. Ngay lập tức, 1 pod có tên là nginx-xxxx được tạo ra. 

```sh
root@adk:/home/adk# kubectl run nginx --image=nginx --port=80
```

```sh
root@adk:/home/adk# kubectl get deployments
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx     1         1         1            1           21m


root@adk:/home/adk# kubectl get pods
NAME                    READY     STATUS    RESTARTS   AGE
nginx-158599303-q8p9v   1/1       Running   0          20m
root@adk:/home/adk# 
```

- Expose the deployment to an external IP address
```sh
root@adk:/home/adk# kubectl expose deployment nginx --type=NodePort
```
Chúng ta phải sử dụng `--type=NodePort` vì minikube không hỗ trợ dịch vụ `LoadBalancer`.

```sh
root@adk:/home/adk# kubectl get svc
NAME                    CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
kubernetes              10.0.0.1     <none>        443/TCP        1h
nginx-158599303-q8p9v   10.0.0.7     <nodes>       80:31035/TCP   12m
```

- Kiểm tra dịch vụ nginx
```sh
root@adk:/home/adk# minikube service nginx --url
http://192.168.99.100:31035
```

```sh
root@adk:/home/adk# curl http://192.168.99.100:31035
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