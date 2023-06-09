1. Tao deployment ten deploy-nginx, voi image nginx:1.16 va replica=1 voi record version. Sau do upgrade len version nginx:1.17 voi record version. Sau do roll back ve nginx:1.16. 
2. Backup etcd, dat ten db backup la ten_hoc_vien.db
3. Tao 1 pod ten la non-root-pod su dung image redis:alpine, voi runAsUser: 1000, fsGroup: 2000
4. Cho deployment sau, tim loi sai va fix loi.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
spec:
  replicas: 5
  selector:
    matchLabels:
      app: hello-world-5
  template:
    metadata:
      labels:
        app: hello-world-5
    spec:
      containers:
      - name: hello-world
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /index.html
            port: 8081
          initialDelaySeconds: 10
          periodSeconds: 10
        resources:
          requests:
            memory: 64M
            cpu: 10m
          limits:
            memory: 64M
            cpu: 10m
---
apiVersion: v1
kind: Service
metadata:
  name: hello-world-5
spec:
  selector:
    app: hello-world
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
```
5. Danh nhan cho node hardware: local_ssd. Tao 1 deployment hello-world-ssd, co volume type emptydir ten redis-storage, mount vao duong dan /data/redis. Pod nay duoc deploy len node co hardware: local_ssd
6. Expose deployment hello-world-ssd voi type NodePort.

Hinh thuc nop bai
- Chup hinh minh chung cac buoc thuc hien vao word.
- 