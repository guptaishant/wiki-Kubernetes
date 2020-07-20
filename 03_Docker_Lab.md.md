# Docker | Application Setup Lab

Let’s get a crash course on playing with Dockerfile and Docker images.

#### Pre-Requisities

- Codebase should be available on your local machine, so we can clone the repo if we haven't done that till now.

```shell
git clone https://github.com/opstree/OT-Microservices-Training.git
```

- **We are using opstree as docker hub account for building the image, we strongly recommend to use your own docker hub account.**

#### MySQL Setup

So here we will use the existing image of MySQL as a base image. To read more about this component please check [README.md](https://github.com/opstree/OT-Microservices-Training/tree/master/mysql) of the application.

```dockerfile
FROM mysql:5.6
ENV MYSQL_ROOT_PASSWORD "password"
ENV MYSQL_DATABASE "attendancedb"
```

Once the Dockerfile for MySQL is created we can create the image from it.

```shell
$ docker build -t opstree/mysql:1.0 -f Dockerfile .
```

In this command, we are providing the image name and tag using `-t` flag and path of Dockerfile using `-f` flag.

Since the IP of the container is dynamic, we need a domain-based system. For that, we can create a docker network and connect the MySQL container to that network.

```shell
docker network create ot-microservice-network -d bridge
```

Once the MySQL image is created we can create the container from that image.

```shell
docker run -itd --name empms-db -p 3306:3306 --net ot-microservice-network opstree/mysql:1.0
```

Let's connect the MySQL with this network:-


We can validate the MySQL by going inside the container:-

```shell
docker exec -it empms-db bash

mysql -u root -ppassword
```

```sql
SHOW DATABASES;
```

#### Attendance Setup

So MySQL service is up and running, so now we can build and deploy the attendance service. To read more about this component please check [README.md](https://github.com/opstree/OT-Microservices-Training/tree/master/attendance) of the application.

```shell
cd attendance
docker build -t opstree/attendance:1.0 -f Dockerfile .
```

Once the attendance image is built, we can run the container from it and connect it to the network

```shell
docker run -d --name empms-attendance -p 8081:8081 --net ot-microservice-network opstree/attendance:1.0
```

#### Gateway Setup

The frontend uses the gateway to communicate with all services, so we need to set up the gateway service before we can hop to frontend service. To read more about this component please check [README.md](https://github.com/opstree/OT-Microservices-Training/tree/master/gateway) of the application.

```shell
cd gateway
docker build -t opstree/gateway:1.0 -f Dockerfile .
```

Now let's run the container from this image

```shell
docker run -d --name empms-gateway -p 8080:8080 --net ot-microservice-network opstree/gateway:1.0
```

#### Frontend Setup

So gateway application is up, now we can deploy the frontend application. To read more about this component please check [README.md](https://github.com/opstree/OT-Microservices-Training/tree/master/frontend) of the application.

```shell
cd frontend
docker build -t opstree/frontend:1.0 -f Dockerfile .
```

Let's run the frontend container from this image

```shell
docker run -d --name empms-frontend -p 5000:5000 --net ot-microservice-network opstree/frontend:1.0
```

#### Webserver Setup

For accessing the frontend application and manage its routes, we have to set up or web server as well. To read more about this component please check [README.md](https://github.com/opstree/OT-Microservices-Training/tree/master/webserver) of the application.

```shell
cd webserver
docker build -t opstree/webserver:1.0 -f Dockerfile .
```

Let's run the webserver from this image

```shell
docker run -d --name empms-webserver -p 80:80 --net ot-microservice-network opstree/webserver:1.0
```

Once the webserver is up, we can validate the attendance functionality from the frontend.

The application UI will look like this:-

<div align="center">
  <img src="https://github.com/opstree/OT-Microservices-Training/raw/master/static/frontend.png">
</div>

## Good To Do

So in the above steps, we enabled the attendance functionality of the application, but other functionalities are still not active because we haven't deployed the other microservice/components.

For deploying the other microservice first thing we need to deploy is elasticsearch as database.

#### Elasticsearch Setup

Let's create a file with the name `elasticsearch.yml` which will have some elasticsearch related configurations. To read more about this component please check [README.md](https://github.com/opstree/OT-Microservices-Training/tree/master/elasticsearch) of the application.

```yaml
---
cluster.name: "ot-microservices"
network.host: 0.0.0.0

discovery.type: single-node

xpack.license.self_generated.type: trial
xpack.security.enabled: true
xpack.monitoring.collection.enabled: true
```

Once the file is created, now we can create the Dockerfile for it.

```dockerfile
# FROM is used for the base image
FROM docker.elastic.co/elasticsearch/elasticsearch:7.5.0

# ENV sets the environment variable for image
ENV ES_JAVA_OPTS "-Xms512m -Xmx512m"

ENV ELASTIC_PASSWORD "elastic"

# COPY can be used to copy stuff from the host system to image
COPY ./elasticsearch.yml /usr/share/elasticsearch/config/elasticsearch.yml
```

All the things are already available in [elasticsearch]() directory as well.

Now we can build the image for elasticsearch.

```shell
docker build -t opstree/elastic:1.0 -f Dockerfile .
```

Once the image is ready, we can set up the elasticsearch

```shell
docker run -d --name empms-es -p 9200:9200 --net ot-microservice-network opstree/elastic:1.0
```

#### Salary Setup

So the elasticsearch application setup is done, now we can onboard the salary application. To read more about this component please check [README.md](https://github.com/opstree/OT-Microservices-Training/tree/master/salary) of the application.

```shell
cd salary
docker build -t opstree/salary:1.0 -f Dockerfile .
```

Let's create a salary application from this image.

```shell
docker run -d --name empms-salary -p 8082:8082 --net ot-microservice-network opstree/salary:1.0
```

#### Employee Setup

After the salary service, we can set up the employee application. To read more about this component please check [README.md](https://github.com/opstree/OT-Microservices-Training/tree/master/employee) of the application.

```shell
cd employee
docker build -t opstree/employee:1.0 -f Dockerfile .
```

Let's deploy the employee application from this image.

```shell
docker run -d --name empms-employee -p 8083:8083 --net ot-microservice-network opstree/employee:1.0
```

Now all applications are on-boarded so we can validate their functionalities from UI.
