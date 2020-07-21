# K8s | Ingress Lab

**Problem Statement:-**

In this lab, we will use ingress to map our internal services to the domain and use a fanout to access our backend. We will have below rules in our ingress:
- <name>-ems.k8slearning.io -> Frontend
- <name>-ems.k8slearning.io/attendance -> Attendance service
- <name>-ems.k8slearning.io/gateway -> Gateway service

By the end of this lab, you will have ingress in place to manage external access.

What you will not have is advanced deployment features such as:
- Ensuring that if your application is ready to function then only k8s sends requests to it.
- If your application has become non-responsive k8s will restart it.
- You can control which nodes your application pod will get deployed. 

**So we are going to use <name> value "tony-stark" in our case. You must change it according to you**

### Important:- We are assuming that you guys have already created the Deployment and Services. If not, please refer here for [Deployment](https://github.com/opstree/OT-Microservices-Training/wiki/09_Deployment_Lab) and [Service](https://github.com/opstree/OT-Microservices-Training/wiki/08_Service_Lab). Make sure you only create Deployment and Service not Replicasets.

## Enable Ingress Minikube

We have to install the ingress in minikube, but it is pretty much straight-forward.

```shell
minikube start --vm=true
minikube addons enable ingress
```

## Frontend

We have to create ingress manifest for frontend with name `frontend-ingress.yaml`

```yaml
apiVersion: networking.k8s.io/v1beta1 # for versions before 1.14 use extensions/v1beta1
kind: Ingress
metadata:
  name: frontend-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: tony-stark.k8slearning.io
    http:
      paths:
      - path: /
        backend:
          serviceName: empms-frontend
          servicePort: 5000
```

Create the frontend ingress resource.

```shell
kubectl apply -f frontend-ingress.yaml
```

Validate the ingress

```shell
kubectl get ingress
```

## Attendance

We have to create ingress manifest for attendance with name `attendance-ingress.yaml`

```yaml
apiVersion: networking.k8s.io/v1beta1 # for versions before 1.14 use extensions/v1beta1
kind: Ingress
metadata:
  name: attendance-ingress
spec:
  rules:
  - host: tony-stark.k8slearning.io
    http:
      paths:
      - path: /attendance
        backend:
          serviceName: empms-attendance
          servicePort: 8081
```

Create the attendance ingress resource.

```shell
kubectl apply -f attendance-ingress.yaml
```

Validate the ingress

```shell
kubectl get ingress
```

## Gateway

We have to create ingress manifest for a gateway with name `gateway-ingress.yaml`

```yaml
apiVersion: networking.k8s.io/v1beta1 # for versions before 1.14 use extensions/v1beta1
kind: Ingress
metadata:
  name: gateway-ingress
spec:
  rules:
  - host: tony-stark.k8slearning.io
    http:
      paths:
      - path: /gateway
        backend:
          serviceName: empms-gateway
          servicePort: 8080
```

Create the gateway ingress resource.

```shell
kubectl apply -f gateway-ingress.yaml
```

Validate the ingress

```shell
kubectl get ingress
```

## Verifying the flow

For verifying the flow, we have to map the Ingress external IP with the domain name which we have provided above.

```shell
minikube ip
```

The output of minikube IP command:-

```shell
172.17.0.3
```

Add this entry in your `/etc/hosts` file.

```
172.17.0.3 tony-stark.k8slearning.io
```
