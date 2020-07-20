# K8s | Kubectl Lab

Let's try to run an elasticsearch container on Kubernetes.

## Installation

Kubectl is a command-line tool for controlling Kubernetes clusters.

### Linux
Download the latest release with the command:
```shell
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```

Make the kubectl binary executable.
```shell
chmod +x ./kubectl
```

Move the binary into your PATH.
```shell
sudo mv ./kubectl /usr/local/bin/kubectl
```

### macOS

Download the latest release with the command:
```shell
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl"
```

Make the kubectl binary executable.
```shell
chmod +x ./kubectl
```

Move the binary into your PATH.
```shell
sudo mv ./kubectl /usr/local/bin/kubectl
```

#### Run Command

```shell
kubectl run elasticsearch --image=opstree/empms-es:1.0 --replicas=1
```

In this command, we are using the run command to create an elasticsearch container on Kubernetes.

- `--image` flag is used to provide the name of the image
- `--replicas` is the count of container which will run

#### Logs and Get-Command

Let's validate the elasticsearch by reading its stdout logs, but for that, we have to get the name of the container, because Kubernetes appends its own id with the container as well.

To list the containers, we can use this command:-

```shell
kubectl get pods
```

Once we get the name of the pod/container we can read their logs:-

```shell
kubectl logs elasticsearch-xxxx
```

#### Exec Command

Let's try to go inside the running container of elasticsearch and validate its functionality.

```shell
kubectl exec -it elasticsearch-xxxx bash
```

Just like docker, we will use `exec` with `it` mode to access the container

Let's put a curl request to validate it.

```shell
curl -XGET -u elastic:elastic http://localhost:9200
```

#### Delete Command

Let's delete the complete elasticsearch setup

```shell
kubectl delete deployment elasticsearch
```

## MySQL Setup

Now let's try to setup MySQL on Kubernetes with kubectl client

```shell
kubectl run mysql --image=opstree/empms-db:1.0 --replicas=1
```

```shell
kubectl get pods
```

#### CP Command

Let's download a dummy SQL file from [here](http://www.sample-videos.com/sql/Sample-SQL-File-10rows.sql)

Once the SQL file is downloaded, we can copy that file to MySQL container

```shell
kubectl cp Sample-SQL-File-10rows.sql mysql-xxxxx:/tmp/
```

Let's try to create the table using this SQL file

```shell
kubectl exec -it mysql-xxxx bash
```

```shell
cd /tmp
mysql -u root -p attendancedb < Sample-SQL-File-10rows.sql
```

#### Port-Forward Command

Now let's try to access MySQL from localhost

```shell
kubectl port-forward mysql-xxxxx 3306:3306
```

Try to connect to MySQL from the host machine

```shell
mysql -u root -p
```

Let's delete the setup

```shell
kubectl delete deployment mysql
```
