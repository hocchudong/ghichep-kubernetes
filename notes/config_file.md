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

## 3. File Service

# Reference
- https://kubernetes.io/docs/reference/api-overview/
- https://kubernetes.io/docs/api-reference/v1.6