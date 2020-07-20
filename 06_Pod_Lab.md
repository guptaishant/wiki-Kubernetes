# K8s | Pod Lab

**Problem Statement:-**

In this lab, we will run MySQL and attendance MS in our k8s cluster using the pod resource of Kubernetes

By the end of this lab you will be able to run your MS using the first resource or kind of k8s i.e Pod.

**What you will not have is auto-recovery of your MS if the pod crashes due to any reason you have to manually create the pod again.**

## MySQL Setup

Let's design and create a Pod manifest for MySQL with the name of `mysql-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: empms-db
spec:
  containers:
  - name: mysql
    image: opstree/empms-db:1.0
    env:
    - name: MYSQL_DATABASE
      value: attendancedb
  restartPolicy: Always
```

Once the manifest is ready let's create a resource from it.

```shell
kubectl apply -f mysql-pod.yaml
```

Verify it by listing pod

```shell
kubectl get pods
```

## Attendance Setup

Let's design and create a Pod manifest for Attendance with the name of `attendance-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: empms-attendance
spec:
  initContainers:
  - name: init-mysql
    image: busybox:1.28
    imagePullPolicy: Always
    command: ['sh', '-c', 'echo The app is running! && sleep 20']
  containers:
  - name: attendance
    image: opstree/empms-attendance:1.0
  restartPolicy: Always
```

Once the manifest is ready let's create a resource from it.

```shell
kubectl apply -f attendance-pod.yaml
```

Verify it by listing pod

```shell
kubectl get pods
```
