# Viết file cấu hình tạo Deployments, pod, services.


## 1. Ví dụ file Pods.
```sh
---
apiVersion: v1
kind: Pod
metadata:
  name: rss-site
  labels:
    app: web
spec:
  containers:
  – name: rss-reader
    image: nginx:1.10
    ports:
      – containerPort: 80
    volumeMounts:
      - name: rss-volume
        mountPath: /var/www/html
  volumes:
  - name: rss-volume
    hostPath:
      path: /data
```

- `apiVersion: v1:` Chỉ ra phiên bản api đang sử dụng. Hiện tại k8s có pod ở phiên bản 1
- `kind: Pod: `: Chỉ loại file mà ta muốn tạo. Có thể là pod, Deployment, Job, Service,.... Tùy theo từng loại mà có các phiên bản api khác nhau. Xem thêm tại đây: https://kubernetes.io/docs/api-reference/v1.6/
- `metadata:`: Là các siêu dữ liệu khi tạo pod.
  - `name: rss-site`: Khai báo tên của pod.
  - `app: web`: Ta bổ sung thêm nhãn `label` với key là `app` và value tương ứng là `web`.
- `spec:`: Khai báo các đối tượng mà k8s sẽ xử lý như: containers, volume,...
  - `containers`: Khai báo đối tượng containers
    - `name: rss-reader`: Tên containers.
    - `image: nginx:1.10`: containers sẽ được build từ image nginx với tag là 1.10
    - `ports:` Liệt kê các port sẽ được containers `expose` ra. Ở đây là port 80.
    - `volumeMounts`: Chỉ định volume sẽ được mount vào containers.
      - `name: rss-volume`: Tên volume.
      - `mountPath: /var/www/html`: Volume sẽ được mount vào thư mục `/var/www/html` trên containers

  - `volumes:`: Khai báo đối tượng volume.
    - `name: rss-volume`: Tên volume
    - `hostPath:`: Volume sẽ sử dụng thư mục `/data` trên máy host.


**Ở trên, tôi chỉ giới thiệu các thuộc tính đặc trưng. Để hiểu rõ hơn, các bạn vào đọc tài liệu api của k8s. Tài liệu được viết rất rõ ràng và chi tiết tại địa chỉ https://kubernetes.io/docs/api-reference/v1.6/**

## 2. File Deployments
```sh
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
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

- `apiVersion: apps/v1beta1`: Chỉ ra phiên bản api sử dụng.
- `kind: Deployment`: Loại file, ở đây là Deployment.
- `metadata:`: Các thông tin bổ sung cho deployment này.
  - `name: nginx-deployment`: Tên deployment.
- `spec:` Khai báo các đối tượng mà k8s sẽ xử lý như: containers, volume,...
  - `replicas: 3`: 3Pods sẽ được tạo ra.
  - `template`: Template định nghĩ pod sẽ được tạo từ template này. Các thông tin trong mục template tương ứng với phần định nghĩa ở File pod ở trên. Tạo ra containers từ image nào, xuất ra port nào, có các labels nào,...

## 3. File Service

```sh
apiVersion: v1
kind: Service
metadata:
  name: dbfrontend
  labels:
    name: dbfrontend
  spec:
    # label keys and values that must match in order to receive traffic for this service
    selector:
      name: dbfrontend
    ports:
     # the port that this service should serve on
      - port: 5555
        targetPort: 3306
    type: NodePort
```

Ta chỉ tập trung vào các thông số:
- `selector:`
  - `name: dbfrontend`: Định tuyến traffic đến pod với cặp giá trị này (name->dbfrontend).
- `port`:
  - `port: 5555`: port sẽ được expose bởi service.
  - `targetPort: 3306`: Port mà truy cập vào pods. (Giá trị từ 1-65535). (port của service trên container).
- `type: NodePort`: type determines how the Service is exposed. Defaults to ClusterIP. NodePort sẽ expose port 5555 trên tất cả các node (cả cụm cluster).


###### Giải thích thêm các thông số trong type:
Trong type có 3 dạng đó là: 

- ClusterIP: "ClusterIP" allocates a cluster-internal IP address for load-balancing to endpoints. Endpoints are determined by the selector or if that is not specified, by manual construction of an Endpoints object. If clusterIP is "None", no virtual IP is allocated and the endpoints are published as a set of endpoints rather than a stable IP. 

- NodePort: on top of having a cluster-internal IP, expose the service on a port on each node of the cluster (the same port on each node). You'll be able to contact the service on any NodeIP:NodePort address.

- LoadBalancer: on top of having a cluster-internal IP and exposing service on a NodePort also, ask the cloud provider for a load balancer which forwards to the Service exposed as a NodeIP:NodePort for each Node.

# Reference
- https://kubernetes.io/docs/reference/api-overview/
- https://kubernetes.io/docs/api-reference/v1.6