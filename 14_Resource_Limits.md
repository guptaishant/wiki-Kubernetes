# K8s | Request and Limits Lab

In this lab we will migrate our MS to a dev namespace so that we can limit them in terms of cluster resources usage:
- Create a namespace dev
- Put resource limit on your Namespace
    - CPU: 500 - 1000 millicore
    - Memory: 500 - 1500 MB
    - Restrict this namespace to not have more than 10 pods
- Deploy your MS
    - MySQL: 200 millicore, (100-500) MB memory
    - Attendance: 100 millicore, 100 MB memory
    - Gateway :  (100-200) millicore, (100-500) MB memory

## Namespace Creation

Create namespace manifest

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: stark-industry
```

```shell
kubectl apply -f namespace.yaml
```

## Limits on Namespace

Let's create a resource quota manifest for our namespace

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
  namespace: stark-industry
spec:
  hard:
    requests.cpu: 500m
    requests.memory: 500Mi
    limits.cpu: "1"
    limits.memory: 1500Mi
    pods: "10"
```

```shell
kubectl apply -f resource-quota.yaml
```

Verify it by

```shell
kubectl describe ns stark-industry
```

## MySQL

Update MySQL Deployment manifest.

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: empms-db
  name: empms-db
  namespace: stark-industry
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
        env:
        - name: MYSQL_DATABASE
          value: attendancedb
        ports:
        - containerPort: 3306
        resources:
          requests:
            memory: "100Mi"
            cpu: "200m"
          limits:
            memory: "500Mi"
            cpu: "200m"
```

Create a resource from this manifest.

```shell
kubectl apply -f mysql-deployment.yaml
```

Validate the service

```shell
kubectl get deployments -n stark-industry
kubectl get pods -n stark-industry
```

## Attendance

Update attendance Deployment manifest.

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: empms-attendance
  name: empms-attendance
  namespace: stark-industry
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
      containers:
      - image: opstree/empms-attendance:1.0
        imagePullPolicy: Always
        name: empms-attendance
        ports:
        - containerPort: 8081
        resources:
          requests:
            memory: "100Mi"
            cpu: "100m"
          limits:
            memory: "100Mi"
            cpu: "100m"
```

Create a resource from this manifest.

```shell
kubectl apply -f attendance-deployment.yaml
```

Validate the service

```shell
kubectl get deployments -n stark-industry
kubectl get pods -n stark-industry
```

## Gateway

Update gateway Deployment manifest

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: empms-gateway
  name: empms-gateway
  namespace: stark-industry
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
        resources:
          requests:
            memory: "100Mi"
            cpu: "100m"
          limits:
            memory: "500Mi"
            cpu: "200m"
```

Create a resource from this manifest.

```shell
kubectl apply -f gateway-deployment.yaml
```

Validate the service

```shell
kubectl get deployments -n stark-industry
kubectl get pods -n stark-industry
```
