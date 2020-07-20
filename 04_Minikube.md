# K8s | Minikube

Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster inside a Virtual Machine (VM) on your laptop for users looking to try out Kubernetes or develop with it day-to-day.

Make sure you have VirtualBox installed.
(Download & Install VirtualBox) : https://www.virtualbox.org/wiki/Downloads

### Linux

Download Minikube:
```shell
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
```

Install Minikube:
```shell
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/
```

Confirm Installation by checking minikube version:
```shell
minikube version
```

### macOS

Download Minikube:
```shell
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 && chmod +x minikube
```

Install Minikube:
```shell
sudo mv minikube /usr/local/bin
```

Confirm Installation by checking minikube version:
```shell
minikube version
```

#### Start K8s cluster using minikube

Start K8s Cluster:
```shell
minikube start
```

Once minukube start finished, check the health of the cluster by executing below command:
```shell
minikube status
```

To stop minikube:
```shell
minikube stop
```

Finally, destroy the k8s cluster:
```shell
minikube delete
```