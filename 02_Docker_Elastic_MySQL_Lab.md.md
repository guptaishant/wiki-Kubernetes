# Docker | Elasticsearch & MySQL Lab

Let’s get a crash course on spinning up Docker containers for Elasticsearch & MySQL.
We will be doing below operations:

## Elasticsearch

- Run a container of the Elasticsearch image on our local machines.

```shell
$ docker run -d --name elasticsearch -e "ELASTIC_PASSWORD=elastic" \
    -e "discovery.type=single-node" -p 9200:9200 \
    docker.elastic.co/elasticsearch/elasticsearch:7.7.0
```

In the above command, we are using `-d` flag to run elasticsearch in detached mode, also we are passing the environment variables using `-e` flag. 
For port mapping we are using `-p` flag through which we are mapping the 9200 port of container to host system.

- Validate if Elasticsearch container is running fine via logs.

```shell
$ docker logs -f elasticsearch
```

We will use `logs` command with docker to check container logs, `-f` flag will be used to follow the logs continuously. `elasticsearch` is the name of ES container

- Try to access Elasticsearch from the host system.

```shell
$ curl -u elastic:elastic http://localhost:9200
```

- Publish image in your Dockerhub account

```shell
$ docker tag docker.elastic.co/elasticsearch/elasticsearch:7.7.0 \
    <your_account>/elasticsearch:7.7.0
```

`tag` command is used to create a docker image as per your docker hub account.


```shell
$ docker login -u <your_account> -p

$ docker push <your_account>/elasticsearch:7.7.0
```

In this command, we are using `docker login` in the Dockerhub registry from cli and then we are using the `push` command to push the image to the registry.

## MySQL

- Run a container of MySQL image on your local machine

```shell
$ docker run -d --name mysql -e MYSQL_ROOT_PASSWORD="password" \
    -e MYSQL_DATABASE="attendancedb" mysql:5.6
```

In the above command, we are trying to run the MySQL docker container in detached mode.

- Create a table in your database

```shell
$ docker exec -it mysql bash

$ mysql -u root -p
```

In this step, we are trying to access the shell of the container by using `exec` command.

Download the SQL file from [here](https://sp.mysqltutorial.org/wp-content/uploads/2018/03/mysqlsampledatabase.zip)

Once the SQL file is downloaded, we can copy that file to MySQL container

```shell
docker cp Sample-SQL-File-10rows.sql mysql:/tmp

docker exec -it mysql bash
```

```shell
cd /tmp
mysql -u root -ppassword attendancedb < Sample-SQL-File-10rows.sql
```

- Start/Stop your container and ensure that data will be retained

```shell
$ docker stop mysql
$ docker start mysql
```

`stop` and `start` command is get used to start and stop the container respectively.

```shell
$ docker tag mysql:5.6 <your_account>/mysql:5.6

$ docker push <your_account>/mysql:5.6
```

In this command, we are using `docker login` from cli and then we are using the `push` command to push the image to the registry.


## Removing Container

Now try to remove the MySQL container.

```shell
docker rm -f MySQL
```

You can create the container again and validate if the data still persists or not.

```shell
$ docker run -d --name mysql -e MYSQL_ROOT_PASSWORD="password" \
    -e MYSQL_DATABASE="attendancedb" mysql:5.6
```

## Good To Do

Mount the volume in MySQL container, in case MySQL container is killed or removed, so data will still persist.

```shell
$ docker run -d --name mysql -e MYSQL_ROOT_PASSWORD="password" \
    -e MYSQL_DATABASE="attendancedb" -v ${PWD}:/var/lib/mysql mysql:5.6
```
