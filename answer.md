###### Answer
```
kubectl create deployment nginx-deploy --image=nginx:1.16 --port 80 --replicas 2 --dry-run=client -o yaml > nginx-deployment.yml

kubectl apply -f nginx-deployment.yml --record

kubectl describe deployment nginx-deploy

kubectl set image deployment nginx-deploy nginx=1.17 --record

kubectl describe deployment nginx-deploy

# rollback to previous 
kubectl rollout history deployment nginx-deploy

kubectl rollout undo deployment nginx-deploy --to-revision=<number_revision_roll_back>

```
```
# Backup etcd in master node
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/backup/etcd.db
```

```
## Fix errors
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
            port: 8080
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
    app: hello-world-5
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
```

```
kubectl get node -o=jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/output/yourname.txt
```

```
Static pod
kubectl run static-busybox --image=busybox --dry-run=client -o yaml --command -- sleep 1000 > static-busybox.yml

mv static-busybox.yml /etc/kubernetes/manifest
```

###### Persistent volume
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-storage
spec:
  capacity:
    storage: 100Mi
  accessModes:
  - ReadWriteMany
  hostPath:
    path: /pv/pv-storage
```

##### Multi-container
```
kubectl run multi-container --image nginx --dry-run=client -o yaml > multi-container.yml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multi-container
  name: multi-container
spec:
  containers:
  - image: nginx
    name: alpha
    env:
      - name: "name"
        value: "alpha"
  - image: busybox
    name: beta
    command:
      - sleep
      - "4800"
    env:
      - name: "name"
        value: "beta"
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

kubectl exec -it non-root-pod -- sh
id
```