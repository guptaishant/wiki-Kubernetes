<div align="center">
  <img src="https://github.com/opstree/OT-Microservices-Training/raw/master/static/logo.svg" height="249" width="255">
  <h1>OT-Microservices</h1>
</div>

Loaded microservice applications for doing different kinds of POC. This is a sample Employee System which is designed to understand the microservices nature and behavior.

## Purpose

The purpose of creating this application is to provide an individual, a holistic idea of microservice architecture, it's working, and its setup.

## Preliminary

- Please work in a group
    - If we are remote, we have break out rooms!
    - We will be running a lot of instances
    - Dev-ops is maddening
- We will let a fun service decide your teamname: http://creativityforyou.com/combomaker.html
- Select a leader of your group, they will need a public Github account and generate a token
- The leader should provide a token to the administrator, via virtual chat, slack, or email

## Applications

These are a few microservices that are getting used in this complete project.

|**Application Name**|**Default Port**|**Dependency**|**Language**|**Description**|
|--------------------|----------------|------------|--------------|---------------|
| [attendance](./attendance) | 8081 | MySQL | <img src="https://cdn.worldvectorlogo.com/logos/gopher.svg" height="32" width="32"> | Attendance is a microservice which is designed in Golang to manage employee's attendance information. |
| [employee](./employee) | 8083 | Elasticsearch | <img src="https://cdn.worldvectorlogo.com/logos/gopher.svg" height="32" width="32"> | Employee microservice is also designed in Golang to manage employee's information. |
| [salary](./salary) | 8082 | Elasticsearch | <img src="https://cdn.worldvectorlogo.com/logos/gopher.svg" height="32" width="32"> | Salary is also a golang based application which creates and manages employee's salary information. |
| [notification](./notification) | - | SMTP Server | <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/c3/Python-logo-notext.svg/1200px-Python-logo-notext.svg.png" height="32" width="32"> | Notification is a scheduled service which gets used to send mail notifications to employees. |
| [frontend](./frontend) | 5000 | Gateway | <img src="https://www.vectorlogo.zone/logos/reactjs/reactjs-icon.svg" height="32" width="32"> | Frontend is written in ReactJS and gets served using nginx proxy, config can be found [here](./webserver) |
| [gateway](./gateway) | 8080 | <ul><li>attendance</li><li>employee</li><li>salary</li><li>notification</li></ul> | <img src="https://www.vectorlogo.zone/logos/java/java-icon.svg" height="32" width="32"> | Gateway is a springboot based API gateway which manages the routing between applications. |
| [webserver](./webserver) | 80 | frontend | <img src="https://cdn.worldvectorlogo.com/logos/gopher.svg" height="32" width="32"> | Webserver is a nginx based proxy which proxies the frontend application. |

For further information about the component, you can click on the application.

## Databases

These applications are using two kinds of databases, one is structured and the other is non-structured.

|**Application Name**|**Default Port**|**Dependency**|**Description**|
|--------------------|----------------|--------------|---------------|
| [elastic](./elastic) | 9092 | - | Elasticsearch is being used as a non-structured database which manages the employee's information and salary. |
| [mysql](./mysql) | 3306 | - | MySQL is getting used for a structured database which manages the employee's attendance information. |

For further information, click on the DB.

## Architecture

The architecture of the complete microservice interaction looks like this:-

<div align="center">
  <img src="https://github.com/opstree/OT-Microservices-Training/raw/master/static/architecture.png">
</div>

## Screenshot

<div align="center">
  <img src="https://github.com/opstree/OT-Microservices-Training/raw/master/static/frontend.png">
</div>
