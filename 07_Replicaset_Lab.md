# K8s | Replicaset Lab

**Problem Statement:-**

In this lab we will try to see the failover/recovery capability of replicaset:
- Delete the previously created pods of MySQL & attendance
- Create a replica set of MySQL having 1 replica
    - Pass database name as an env variable
    - Provide port 3306 as a container port
- Create a replica set of attendance having 2 replica
    - Provide port 8081 as a container port
- Create a replica set of gateway having 1 replica
- Try deleting pods of MySQL & attendance MS

**What you will not have is connectivity will not be in place between the MS also you won’t be able to expose your MS for public consumption.**

## Deleting previous Pods

```shell
kubectl delete pod empms-db
kubectl delete pod empms-attendance
```

## MySQL Setup

Let's create a MySQL replicaset manifest with name `mysql-replicaset.yaml`

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: empms-db
  labels:
    app: empms-db
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
      - name: empms-db
        image: opstree/empms-db:1.0
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_DATABASE
          value: "attendancedb"
```

Let's create the resource in Kubernetes

```shell
kubectl apply -f mysql-replicaset.yaml
```

## Attendance Setup

Let's try to setup attendance with replicaset with name `attendance-replicaset.yaml`

Firstly, we have to design a replicaset manifest for the same.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: empms-attendance
  labels:
    app: empms-attendance
spec:
  replicas: 2
  selector:
    matchLabels:
      app: empms-attendance
  template:
    metadata:
      labels:
        app: empms-attendance
    spec:
      containers:
      - name: empms-attendance
        image: opstree/empms-attendance:1.0
        ports:
        - containerPort: 8081
```

Let's create the resource in Kubernetes

```shell
kubectl apply -f attendance-replicaset.yaml
```

## Gateway Setup

Let's try to setup gateway with replicaset with name `gateway-replicaset.yaml`

Firstly, we have to design a replicaset manifest for the same.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: empms-gateway
  labels:
    app: empms-gateway
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
      - name: empms-gateway
        image: opstree/empms-gateway:1.0
        ports:
        - containerPort: 8080
```

Let's create the resource in Kubernetes

```shell
kubectl apply -f gateway-replicaset.yaml
```

## Validate Services

Let's validate the attendance service.

List out the pod for attendance MS service

```shell
kubectl get pods -l app=empms-attendance
```

**Please Note:- We are not listing all pods, we are searching by this label `app=empms-attendance`**

The output of kubectl get pods command:-

```shell
NAME                     READY   STATUS    RESTARTS   AGE
empms-attendance-n88z5   1/1     Running   0          65s
```

```shell
kubectl exec -it empms-attendance-xxx sh
```

```shell
curl -XGET http://localhost:8081/attendance/healthz
```

Exit from the pod and validate the logs of the service.

```shell
kubectl logs empms-attendance-xxx
```

## Deleting Replicaset Pods

```shell
kubectl get pods
```

```shell
kubectl delete pod empms-attendance-xxxx
kubectl delete pod empms-db-xxxx
```

## Connectivity Test

Let's test the connectivity between the pods:-

```shell
kubectl exec -it empms-attendance sh
```

```shell
curl -XGET empms-gateway:8080
```

You will see that we are not able to curl the gateway service from attendance because we haven't configured the services yet.
