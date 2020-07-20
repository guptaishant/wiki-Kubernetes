# K8s | Deployment Lab

**Problem Statement:-**

In this lab we will deploy a few of our microservices in the Kubernetes cluster using Kubernetes Deployment:

- MySQL
- Attendance
- Gateway
- Frontend
- Webserver

By the end of this lab, you should be able to have these individual MS up and running.

**What you will not have is the persistence of your MySQL database i.e. if your MySQL pod is restarted you will lose complete data.** 

## Deleting Old Replica sets

Before going to the deployment we have to clean up the old replica sets.

```shell
kubectl delete rs empms-db empms-attendance empms-gateway empms-frontend empms-webserver
```

## Deployments

#### MySQL Deployment

Create a MySQL Deployment manifest with name `mysql-deployment.yaml`

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
        env:
        - name: MYSQL_DATABASE
          value: attendancedb
        ports:
        - containerPort: 3306
```

Create a resource from this manifest.

```shell
kubectl apply -f mysql-deployment.yaml
```

Validate the service

```shell
kubectl get deployments
kubectl get pods
```

#### Attendance

Create an attendance Deployment manifest with name `attendance-deployment.yaml`

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
      containers:
      - image: opstree/empms-attendance:1.0
        imagePullPolicy: Always
        name: empms-attendance
        ports:
        - containerPort: 8081
```

Create a resource from this manifest.

```shell
kubectl apply -f attendance-deployment.yaml
```

Validate the service

```shell
kubectl get deployments
kubectl get pods
```

#### Gateway

Create a gateway Deployment manifest with name `gateway-deployment.yaml`

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
```

Create a resource from this manifest.

```shell
kubectl apply -f gateway-deployment.yaml
```

Validate the service

```shell
kubectl get deployments
kubectl get pods
```

#### Frontend

Create a frontend Deployment manifest with name `frontend-deployment.yaml`

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: empms-frontend
  name: empms-frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: empms-frontend
  template:
    metadata:
      labels:
        app: empms-frontend
    spec:
      containers:
      - image: opstree/empms-frontend:1.0
        imagePullPolicy: Always
        name: empms-frontend
        ports:
        - containerPort: 5000
```

Create a resource from this manifest.

```shell
kubectl apply -f frontend-deployment.yaml
```

Validate the service

```shell
kubectl get deployments
kubectl get pods
```

#### Webserver

Create a webserver Deployment manifest with name `webserver-deployment.yaml`

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: empms-webserver
  name: empms-webserver
spec:
  replicas: 1
  selector:
    matchLabels:
      app: empms-webserver
  template:
    metadata:
      labels:
        app: empms-webserver
    spec:
      containers:
      - image: opstree/empms-webserver:1.0
        imagePullPolicy: Always
        name: empms-webserver
        ports:
        - containerPort: 80
```

Create a resource from this manifest.

```shell
kubectl apply -f webserver-deployment.yaml
```

Validate the service

```shell
kubectl get deployments
kubectl get pods
```

## Attendance V2

So if you try to list the attendance from UI there will be a blank page, which means attendance listing functionality is not working. So we can update the deployment to use the version of the image.

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: empms-attendance
  name: empms-attendance
spec:
  replicas: 5
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
      # Changing the tag from 1.0 to 2.0
      - image: opstree/empms-attendance:2.0
        imagePullPolicy: Always
        name: empms-attendance
        ports:
        - containerPort: 8081
```

Update the deployment

```shell
kubectl apply -f attendance-deployment.yaml
```

Validate the deployment

```shell
kubectl get pods -w
```
