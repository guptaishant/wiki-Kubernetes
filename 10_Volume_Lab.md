# K8s | Volume Lab

**Problem Statement:-**

In this lab we will add volume to our MySQL service and externalize the connectivity details of Attendance & Gateway MS:

- MySQL: Should use hostPath for volume mounting
- Attendance: Should use the secret for MySQL connectivity
- Gateway: Should use ConfigMap for Attendance connectivity
- ES: Use emptyDir 

By the end of this lab, you have partial persistence in place and now you can use the same image to deploy it in various environments as environmental configurations are externalized.

**What you will not have is full statefulness  for the MySQL DB i.e if our MySQL pod gets recreated on some other node you will lose complete data.**

## Host Path

Update the MySQL deployment manifest to use hostPath

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
          hostPath:
            path: /opt
            type: Directory
```

Once the manifest is updated, we can update the DB deployment.

```shell
kubectl apply -f mysql-deployment.yaml
```

## Secret

Create a secret that will have a DB username and password.

```shell
echo -n "root" | base64
```

```shell
echo -n "password" | base64
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: cm9vdA==
  password: cGFzc3dvcmQ=
```

```shell
kubectl apply -f db-secret.yaml
```

Once the secret is created let's update the attendance manifest to use credentials

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: empms-attendance
  name: empms-attendance
spec:
  replicas: 1
  selector:
    matchLabels:
      app: empms-attendance
  template:
    metadata:
      labels:
        app: empms-attendance
    spec:
      initContainers:
      - name: init-mysql
        image: opstree/empms-mysql-healthcheck:1.0
        imagePullPolicy: Always
        env:
        - name: MYSQL_USERNAME
          value: "root"
        - name: MYSQL_PASSWORD
          value: "password"
        - name: MYSQL_HOST
          value: "empms-db"
        - name: MYSQL_DATABASE
          value: "attendancedb"
        - name: SLEEP_INTERVAL
          value: "30"
      containers:
      - image: opstree/empms-attendance:2.0
        imagePullPolicy: Always
        name: empms-attendance
        ports:
        - containerPort: 8081
        env:
        - name: MYSQL_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
```

After updating the manifest, we can update the deployment of Attendance.


```shell
kubectl apply -f attendance-deployment.yaml
```

## ConfigMap

Create a config map manifest for gateway service with the name `gateway-configmap.yaml`.

```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: gateway-config
data:
  application.yml: |
    spring:
      profiles: default

    eureka:
    client:
      healthcheck:
        enabled: true

    zuul:
    debug:
      request: true

    routes:
      employee:
      path: /employee/**
      url: http://empms-employee:8083/employee
      service-id: /employee/**

      attendance:
      path: /attendance/**
      url: http://empms-attendance:8081/attendance
      service-id: /attendance/**

      salary:
      path: /salary/**
      url: http://empms-salary:8082/salary
      service-id: /salary/**
```

```shell
kubectl apply -f gateway-configmap.yaml
```

Once the config map is created, we can update the gateway deployment to use this configmap

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: empms-gateway
  name: empms-gateway
spec:
  replicas: 1
  selector:
    matchLabels:
      app: empms-gateway
  template:
    metadata:
      labels:
        app: empms-gateway
    spec:
      containers:
      - image: opstree/empms-gateway:1.0
        imagePullPolicy: Always
        name: empms-gateway
        ports:
        - containerPort: 8080
        volumeMounts:
        - mountPath: /app/config/application.yml
          name: gateway-config
          subPath: application.yml
      volumes:
        - name: gateway-config
          configMap:
            name: gateway-config
```

Update the gateway deployment

```shell
kubectl apply -f gateway-deployment.yaml
```

## EmptyDir

Create an elasticsearch deployment with emptyDir as volume.

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: empms-es
  name: empms-es
spec:
  replicas: 1
  selector:
    matchLabels:
      app: empms-es
  template:
    metadata:
      labels:
        app: empms-es
    spec:
      containers:
      - image: opstree/empms-es:1.0
        imagePullPolicy: Always
        name: empms-es
        ports:
        - containerPort: 9200
        volumeMounts:
        - name: data
          mountPath: /usr/share/elasticsearch/data
      volumes:
        - name: data
          emptyDir: {}
```

Create the resource for es

```shell
kubectl apply -f elastic-deployment.yaml
```

## Data Validation

Let's try to delete the MySQL pod and see if data persists.

```shell
kubectl delete pod empms-db-xxxx
```

If you are using minikube the data will be persisted because we have a single node. But in multi-node Kubernetes, we will lose the data if pod is created on some other node.
