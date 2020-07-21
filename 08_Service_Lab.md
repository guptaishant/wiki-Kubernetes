# K8s | Service Lab

**Problem Statement:-**

In this lab we will create services on top of our deployed microservices so that they can talk to each other and making our webserver MS publically accessible:

Expose Internally

- MySQL
- Attendance
- Gateway
- Frontend

Expose Publically
- Webserver

By the end of this lab, you should be able to have the attendance functionality fully functional.

**What you will not have is the automated deployment of your MS. To do a new deployment you have to create a new replica set and delete previous one.**

### Frontend Replicaset

Let's try to set up frontend with replicaset with name `frontend-replicaset.yaml`

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: empms-frontend
  labels:
    app: empms-frontend
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
      - name: empms-frontend
        image: opstree/empms-frontend:1.0
        ports:
        - containerPort: 5000
```

Let's create the resource in Kubernetes

```shell
kubectl apply -f frontend-replicaset.yaml
```

### Webserver Replicaset

Let's try to set up a webserver with replicaset with name `webserver-replicaset.yaml`

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: empms-webserver
  labels:
    app: empms-webserver
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
      - name: empms-webserver
        image: opstree/empms-webserver:1.0
        ports:
        - containerPort: 80
```

Let's create the resource in Kubernetes

```shell
kubectl apply -f webserver-replicaset.yaml
```

**Note:- If you see webserver pod will be in an error state because it cannot resolve the name of frontend service. It will be fixed once we create the service for Frontend application.**

## Service Setup

Let's create the service for all applications.

#### MySQL Service

Create a manifest for MySQL service with name `mysql-service.yaml`

```yaml
---
kind: Service
apiVersion: v1
metadata:
  name: empms-db
spec:
  type: ClusterIP
  selector:
    app: empms-db
  ports:
  - protocol: TCP
    port: 3306
```

Create the service from this manifest.

```shell
kubectl apply -f mysql-service.yaml
```

Now let's verify the service

```shell
kubectl get svc
```

#### Attendance Service

Create a manifest for attendance service with name `attendance-service.yaml`

```yaml
---
kind: Service
apiVersion: v1
metadata:
  name: empms-attendance
spec:
  type: ClusterIP
  selector:
    app: empms-attendance
  ports:
  - protocol: TCP
    port: 8081
```

Create the service from this manifest.

```shell
kubectl apply -f attendance-service.yaml
```

Now let's verify the service

```shell
kubectl get svc
```

#### Gateway Service

Create a manifest for gateway service with name `gateway-service.yaml`

```yaml
---
kind: Service
apiVersion: v1
metadata:
  name: empms-gateway
spec:
  type: ClusterIP
  selector:
    app: empms-gateway
  ports:
  - protocol: TCP
    port: 8080
```

Create the service from this manifest.

```shell
kubectl apply -f gateway-service.yaml
```

Now let's verify the service

```shell
kubectl get svc
```

#### Frontend Service

Create a manifest for frontend service with name `frontend-service.yaml`

```yaml
---
kind: Service
apiVersion: v1
metadata:
  name: empms-frontend
spec:
  type: ClusterIP
  selector:
    app: empms-frontend
  ports:
  - protocol: TCP
    port: 5000
```

Create the service from this manifest.

```shell
kubectl apply -f frontend-service.yaml
```

Now let's verify the service

```shell
kubectl get svc
```

#### Webserver Service

Create a manifest for webserver service with name `webserver-service.yaml`

##### NodePort

```yaml
---
kind: Service
apiVersion: v1
metadata:
  name: empms-webserver
spec:
  type: NodePort
  selector:
    app: empms-webserver
  ports:
  - protocol: TCP
    port: 80
    nodePort: 30085
```

Here we are using the NodePort to expose the service, but it's not a recommended way to expose service in Production.

Create the service from this manifest.

```shell
kubectl apply -f webserver-service.yaml
```

```shell
kubectl get svc
```

We can validate the service by using the NodePort which we have defined above.

##### LoadBalancer

```shell
kubectl delete -f webserver-service.yaml
```

The definition of load balancer type service would be:-

```yaml
---
kind: Service
apiVersion: v1
metadata:
  name: empms-webserver
spec:
  type: LoadBalancer
  selector:
    app: empms-webserver
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    name: http
```

Here we are using the LoadBalancer to expose the service.

Now let's verify the service

```shell
kubectl get svc
```

```shell
minikube service empms-webserver
```

We can use the LoadBalancer public IP to access the application from the web browser.

## Automated Deployment Test

Now try to create the new replicaset with version 2 of the image.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: empms-frontend-version2
  labels:
    app: empms-frontend-version2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: empms-frontend-version2
  template:
    metadata:
      labels:
        app: empms-frontend-version2
    spec:
      containers:
      - name: empms-frontend
        image: opstree/empms-frontend:2.0
        ports:
        - containerPort: 5000
```

Now for a new version of deployment, we have to delete the first replicaset.

```shell
kubectl delete rs emps-frontend
```

And then create again

```shell
kubectl apply -f frontend-replicaset.yaml
```
