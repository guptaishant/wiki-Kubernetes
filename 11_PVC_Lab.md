# K8s | PVC Lab

**Problem Statement:-**

In this lab we will add persistent volume to our MySQL service and will try to recreate without worrying too much on which node it will get created:

By the end of this lab, you have a fully functional durable Attendance functionality.

What you will not have is various advanced features such as:
- Autoscaling
- Deciding which app should get deployed on which node.
- Running a nightly scheduled operation to send emails. 
- Putting boundaries on resources usages

### PVC

First, we need to create the manifest for PV with name `mysql-pv.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

Create the resource in Kubernetes

```shell
kubectl apply -f mysql-pv.yaml
```

Verify by listing the pods

```shell
kubectl get pv
```

Once the PV is created, we can create the PVC to claim it.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-volume
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Create the PVC in Kubernetes

```shell
kubectl apply -f mysql-pvc.yaml
```

### MySQL

Now let's update the MySQL deployment to use this PVC.

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: empms-db
  name: empms-db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: empms-db
  template:
    metadata:
      labels:
        app: empms-db
    spec:
      containers:
      - image: opstree/empms-db:1.0
        imagePullPolicy: Always
        name: empms-db
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_DATABASE
          value: attendancedb
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: db-storage
      volumes:
      - name: db-storage
        persistentVolumeClaim:
          claimName: mysql-volume
```

Once the manifest is updated, let's apply it.

```shell
kubectl apply -f mysql-deployment.yaml
```

Verify the pods

```shell
kubectl get pods
```

## Validation

Let's try to delete and re-create the deployment and see if data persists.

```shell
kubectl delete -f mysql-deployment.yaml
```

```shell
kubectl apply -f mysql-deployment.yaml
```

```shell
kubectl exec -it empms-db-xxx bash
```

```shell
mysql -u root -p
```

```SQL
SHOW DATABASES;
```
