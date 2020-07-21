# K8s | Probes Lab

**Problem Statement:-**

In this lab, we will use liveness and readiness probes to ensure that k8s to handle the availability of microservices. Implement readiness & liveness for all of our microservice:
- Salary, Employee, Attendance, Gateway
- Identify the difference in deployment with and without readiness probe

By the end of this lab, you will have a clear understanding of probes and how important a role they play in a fully functional app.

**What you will not have yet is control in terms of allocation of pods in your k8s cluster**

## Elasticsearch

So now let's deploy the elasticsearch and it's service to make this work.

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
```

```shell
kubectl apply -f elastic-deployment.yaml
```

Once after that, we need to create the service of elasticsearch

```yaml
---
kind: Service
apiVersion: v1
metadata:
  name: empms-es
spec:
  type: ClusterIP
  selector:
    app: empms-es
  ports:
  - protocol: TCP
    port: 9200
```

```shell
kubectl apply -f elastic-service.yaml
```

## Salary

We have to create the salary component with readiness and liveness probe.

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: empms-salary
  name: empms-salary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: empms-salary
  template:
    metadata:
      labels:
        app: empms-salary
    spec:
      containers:
      - image: opstree/empms-salary:1.0
        imagePullPolicy: Always
        name: empms-salary
        env:
        - name: DELAY_TIME
          value: "5"
        ports:
        - containerPort: 8082
          name: client
        livenessProbe:
          httpGet:
            path: /salary/healthz
            port: 8082
          initialDelaySeconds: 10
          periodSeconds: 5
          successThreshold: 1
          timeoutSeconds: 10
        readinessProbe:
          httpGet:
            path: /salary/healthz
            port: 8082
          initialDelaySeconds: 10
          periodSeconds: 5
          successThreshold: 1
          timeoutSeconds: 10
```

Create the salary deployment

```shell
kubectl apply -f salary-deployment.yaml
```

#### Liveness Test

Let's test the liveness for salary application.

Exec inside the salary pod by `kubectl exec`.

```shell
kubectl exec -it empms-salary-xxx bash
```

Once you are inside the salary pod, put this curl request

```shell
curl -XPOST http://localhost:8082/salary/configure/liveness?delay_time=100
```

This will delay the response of health check

**Now if you see that Kubernetes will restart the salary application because of delay in response.**

You can watch it by

```shell
kubectl get pod -w
```

## Employee

Since we have validated and understood the concept of readiness and liveness. We can add it to other applications as well.

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: empms-employee
  name: empms-employee
spec:
  replicas: 1
  selector:
    matchLabels:
      app: empms-employee
  template:
    metadata:
      labels:
        app: empms-employee
    spec:
      containers:
      - image: opstree/empms-employee:1.0
        imagePullPolicy: Always
        name: empms-employee
        ports:
        - containerPort: 8083
        livenessProbe:
          httpGet:
            path: /employee/healthz
            port: 8083
          initialDelaySeconds: 3
          periodSeconds: 3
          successThreshold: 1
          timeoutSeconds: 10
        readinessProbe:
          httpGet:
            path: /employee/healthz
            port: 8083
          initialDelaySeconds: 3
          periodSeconds: 3
          successThreshold: 1
          timeoutSeconds: 10
```

```yaml
---
kind: Service
apiVersion: v1
metadata:
  name: empms-employee
spec:
  type: ClusterIP
  selector:
    app: empms-employee
  ports:
  - protocol: TCP
    port: 8083
```

Now create the deployment and service for employee

```shell
kubectl apply -f employee-deployment.yaml
kubectl apply -f employee-svc.yaml
```

## Attendance

Update the attendance Deployment manifest.

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
        livenessProbe:
          httpGet:
            path: /attendance/healthz
            port: 8081
          initialDelaySeconds: 3
          periodSeconds: 3
          successThreshold: 1
          timeoutSeconds: 10
        readinessProbe:
          httpGet:
            path: /attendance/healthz
            port: 8081
          initialDelaySeconds: 3
          periodSeconds: 3
          successThreshold: 1
          timeoutSeconds: 10
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

**Make sure empms-db is deployed otherwise readiness check will fail. You can refer [deployment](https://github.com/opstree/OT-Microservices-Training/wiki/09_Deployment_Lab) and [service](https://github.com/opstree/OT-Microservices-Training/wiki/08_Service_Lab)**


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
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 3
          periodSeconds: 3
          successThreshold: 1
          timeoutSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 3
          periodSeconds: 3
          successThreshold: 1
          timeoutSeconds: 10
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
