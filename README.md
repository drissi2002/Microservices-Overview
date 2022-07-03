# Microservices-Overview
## Repository objectives
- Building microservices with Spring Boot and Spring Cloud
- Deploying a microservice on multiple instances
## Tools and Versions
- Spring Boot Version:2.6.9
- Spring Cloud Version:2.6.x
- JavaVersion:1.8.0_121
- Maven Version:3.8.6
- IntelliJ IDEA Version Ultimate 2021.3 (or any other IDE of your choice)

## Microservice Architecture
### Presentation
A Microservices architecture represents a way of designing applications as a set of independently deployable services. <br>
These services should preferably be organized around business skills, automatic deployment, intelligent endpoints and decentralized control of technology and data.

### Proposed Architecture
- The objective of this work is to show how to create several independently deployable services that communicate with each other, using the facilities offered by Spring Cloud and Spring Boot. 
- Spring Cloud provides tools for developers to quickly and easily build common patterns of distributed systems (such as configuration, discovery, or intelligent routing services).
- Spring Boot, on the other hand, makes it possible to build Spring applications quickly as quickly as possible, minimizing the usually painful configuration time of Spring applications.

So we will create the following microservices:

1) Product Service: Main service, which offers a REST API to list a list of products.
2) Config Service: Configuration service, whose role is to centralize the configuration files of the different microservices in a single place.
3) Proxy Service: Gateway responsible for routing a request to one of the instances of a service, so as to automatically manage the load distribution.
4) Discovery Service: Service allowing the registration of instances of services in order to be discovered by other services.

The resulting architecture will look like this:
<p align="center">
<img src="https://i.imgur.com/WXY8Pdd.png" title="source: imgur.com" /></p>

## Creating Microservices
### Microservice ProductService
We start by creating the main service: *Product Service*.
<p align="center">
<img src="https://i.imgur.com/9zhEM3t.png" title="source: imgur.com" /></p>

Each microservice will be in the form of a Spring project. To quickly and easily create a Spring project with all necessary dependencies, Spring Boot provides Spring Initializr.<br>
To do this, go to the [*start.spring.io*](https://start.spring.io/) site, and create a project with the following characteristics:

- Maven project with Java and Spring Boot version 1.5.8
- Group: tn.enicarthage.overmicro
- Artifact: product service
- Dependencies:
<p align="center">
<img src="https://i.imgur.com/7t6mcya.png" title="source: imgur.com" /></p>

Then follow these steps to create the ProductService microservice:

1) Open the downloaded project with IntelliJ IDEA.
2) Under the src/main/java directory and in the tn.enicarthage.overmicro.productservice package, create the following Product class:

``
package tn.insat.tpmicro.productservice;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import java.io.Serializable;

@Entity
public class Product implements Serializable {
    @Id
    @GeneratedValue
    private int id;
    private String name;
    public Product(){
    }
    public Product(String name) {
        this.name = name;
    }
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
``




















