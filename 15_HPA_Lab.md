# K8s | HPA Lab

In this lab we will use HPA to ensure that k8s will take care of scaling and de-scaling of pods of your MS on the basis of CPU & Memory usage :

- Enable autoscaling for Attendance MS with a threshold of 50% avg CPU usage of all instances.
- Enable autoscaling for Employee MS with a threshold limit of 50% of avg Memory.

By the end of this lab, you will have to be able to deploy your microservices in such a fashion that they can scale up & down on the basis of CPU/memory usages.

## Pre-requisities

We have to restart minikube to enable some config for HPA

```shell
minikube start --extra-config=controller-manager.horizontal-pod-autoscaler-upscale-delay=1m --extra-config=controller-manager.horizontal-pod-autoscaler-downscale-delay=1m --extra-config=controller-manager.horizontal-pod-autoscaler-sync-period=10s --extra-config=controller-manager.horizontal-pod-autoscaler-downscale-stabilization=1m
```

```shell
minikube addons enable metrics-server
```

## Attendance

So firstly, we have to give resources and limits because HPA uses them to calculate usage and then scale it.

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
      - image: opstree/empms-attendance:2.0
        imagePullPolicy: Always
        name: empms-attendance
        ports:
        - containerPort: 8081
        resources:
          requests:
            memory: "100Mi"
            cpu: "10m"
          limits:
            memory: "100Mi"
            cpu: "10m"
```

```shell
kubectl apply -f attendance-deployment.yaml
```

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

```shell
kubectl apply -f attendance-service.yaml
```

Create an HPA manifest for attendance application

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: empms-attendance-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: empms-attendance
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization 
        averageUtilization: 50
```

```shell
kubectl apply -f attendance-hpa.yaml
```

Validate it by

```shell
kubectl get hpa
```

![](https://github.com/opstree/OT-Microservices-Training/blob/master/static/hpa.png?raw=true)

## Generate Load and Testing

Let's run a pod with the name `load-generator`

```shell
kubectl run -it --rm load-generator --image=busybox /bin/sh
```

Run this command:-

```shell
while true; do wget -q -O- http://empms-attendance:8081/attendance/search; done
```

And wait for 2 to 3 minutes and you will have something like this:-

![](https://github.com/opstree/OT-Microservices-Training/blob/master/static/hpa.png?raw=true)

![](https://github.com/opstree/OT-Microservices-Training/blob/master/static/hpa2.png?raw=true)
