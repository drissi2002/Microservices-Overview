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

```
package tn.enicarthage.overmicro.productservice;

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
```
3) This class is annotated with JPA, to then store the Product objects in the H2 database thanks to Spring Data. To do this, create the ProductRepository interface in the same package:
```
package tn.enicarthage.overmicro.productservice;

import org.springframework.data.jpa.repository.JpaRepository;

public interface ProductRepository extends JpaRepository<Product , Integer> {
}
```

4) To insert the objects into the database, we will use the Stream object. For this, we will create the DummyDataCLR class:
```
package tn.enicarthage.overmicro.productservice;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

import java.util.stream.Stream;

@Component
class DummyDataCLR implements CommandLineRunner {

    @Override
    public void run(String... strings) throws Exception {
        Stream.of("Pencil", "Book", "Eraser").forEach(s->productRepository.save(new Product(s)));
        productRepository.findAll().forEach(s->System.out.println(s.getName()));
    }

    @Autowired
    private ProductRepository productRepository;

}

```
*To insert the objects into the database, we will use the Stream object. For this, we will create the DummyDataCLR class:*

5) Launch the main class. An H2 database will be created and the CommandLineRunner will take care of injecting the data into it.
6) To run your application:
Create a mvn package configuration by doing Run->Edit Configurations then creating a new Maven-like configuration with the package command as follows:
<br>

![image](https://user-images.githubusercontent.com/84160502/177022457-ff43cb08-bc82-4a43-bd69-cc4b43ed05dd.png)

<br>

A target directory will be created, containing the generated classes:<br>

![image](https://user-images.githubusercontent.com/84160502/177022489-61702b2e-09bb-4fa6-b45e-9e52db35ac86.png)

<br>

6) Then launch the Spring Boot *ProductServiceApplication* configuration created by default by IntelliJ. The output on the console should look like the following:
<br>

![image](https://user-images.githubusercontent.com/84160502/177022947-4d14b5ad-ed9e-446e-afc9-7aacf5e95c71.png)

<br>

To test your application, open the http://localhost:8081 page on the browser. You will get (if all goes well) the following result:
<br>

![image](https://user-images.githubusercontent.com/84160502/177022984-53351639-aa17-4b94-b6a5-39ca5e95dcb0.png)

<br>

You will notice that the REST service created automatically respects the HATEOAS standard, which offers in REST services, links to navigate dynamically between interfaces.

If you navigate to the http://localhost:8081/products page, you will see the list of products, injected by the CLR, as follows:
<br>

![image](https://user-images.githubusercontent.com/84160502/177023008-e5b7d0c3-0fe5-4dc2-b930-f437d00ddec1.png)

<br>

To see information about a single product, you just need to know its ID: http://localhost:8081/products/1, for example.
<br>

![image](https://user-images.githubusercontent.com/84160502/177023043-b81182e5-1c29-495b-bcbc-2649f2752ec0.png)

<br>
To add search-by-name functionality, for example, modify the ProductRepository interface, as follows:

```
package tn.enicarthage.overmicro.productservice;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;

@RepositoryRestResource
public interface ProductRepository extends JpaRepository<Product , Integer> {

    @Query("select p from Product p where p.name like :name")
    public Page<Product> productByName(@Param("name") String mc
            , Pageable pageable);
}

```

To test this search functionality, go to the link http://localhost:8080/products/search/productByName?name=Eraser 
The result obtained will be the following:
<br>

![image](https://user-images.githubusercontent.com/84160502/177023227-8f9c88d5-b14b-493d-b14b-e44f68032e19.png)

<br>

The Actuator dependency that has been added to the project allows you to display information about your REST API without having to explicitly implement the functionality. For example, if you go to http://localhost:8081/metrics, you will be able to have several information about the microservice, such as the number of threads, the memory capacity, the class loaded in memory, etc. But first, add the following two lines to the src/main/resources/application.properties file to (1) display more detailed service status information and (2) disable default security constraints:

```
 endpoints.health.sensitive = false
 management.security.enabled = false
```


Les informations sur l'état du service sont affichées grâce à http://localhost:8081/actuator/health : 

<br>

![image](https://user-images.githubusercontent.com/84160502/177023622-45968fad-9eec-42fb-b09b-69f875da1dcd.png)

<br>

### Multiple Instances of the Microservice ProductService
We will now create other instances of the same service and deploy them to different ports.

<p align="center">
<img src="https://i.imgur.com/1Bpk5ly.png" title="source: imgur.com" /></p>

To launch multiple instances of the ProductService service, we will define multiple configurations with different port numbers. 
For it:
- Go to Run->Edit Configurations, and copy the ProductServiceApplication configuration to it selecting in the sidebar, and clicking on the icon <img alt="copy" src="https://i.imgur.com/E6GrFMa.png"> . A new configuration will be created.
- Change its name: ProductServiceApplication:8082
- Add in the Program Arguments box the following argument:
```
--server.port=8082
```

- Start setup. A new service will be available at: http://localhost:8082
<br>

![image](https://user-images.githubusercontent.com/84160502/177055862-ba5d034c-26e7-4cdd-a223-ce5cfd98dacb.png)

<br>
- Repeat the same steps to create an instance of the service running on port 8083.
### Microservice ConfigService

In a microservices architecture, multiple services run at the same time, on different processes, each with its own configuration and settings. Spring Cloud Config provides server-side and client-side support for outsourcing configurations in a distributed system. Thanks to the configuration service, it is possible to have a centralized place to manage the properties of each of these services. 

<p align="center">
<img src="https://i.imgur.com/V3S5aii.png" title="source: imgur.com" /></p>

So:

- Start by creating a ConfigService service in Spring Initializr, with the appropriate dependencies, as shown in the following figure:
<br>

![image](https://user-images.githubusercontent.com/84160502/177057986-2c76084a-4682-464a-a4fa-06ebdb44e8d5.png)

<br>

- Open the project in another instance of IntelliJ IDEA.
- To expose a configuration service, use the @EnableConfigServer annotation for the ConfigServiceApplication class, as follows:
```
package tn.enicarthage.overmicro.configservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@EnableConfigServer
@SpringBootApplication
public class ConfigServiceApplication {

    public static void main(String[] args) {

        SpringApplication.run(ConfigServiceApplication.class, args);
    }
}
```
- To configure this configuration service, add the following values to its application.properties file:
```
server.port=8888
spring.cloud.config.server.git.uri=file:./src/main/resources/myConfig
```
Ceci indique que le service de configuration sera lancé sur le port 8888 et que le répertoire contenant les fichiers de configuration se trouve dans le répertoire src/main/resources/myConfig. Il suffit maintenant de créer ce répertoire.

- Create myConfig directory at src/main/resources tree
- Create in this directory the application.properties file in which you insert the following instruction:
```
global=xxxxx
```
This file will be shared between all microservices using this configuration service.

- The configuration directory must be a git directory. For it:
1) Open the terminal with IntelliJ and navigate to this directory.
2) Initialize your directory: ```git init```
3) Create a root entry in the repository: ```git add .```
4) Make a commit: ```git commit -m "add ." ```

5) Go back to the ProductService project and add in the application.properties configuration file:
```
server:
  port: 8081
spring:
  cloud:
    config:
      enabled: true
  config:
    import: configserver:http://localhost:8888
endpoints.health.sensitive: false
management.security.enabled: false

spring.application.name: product-service

```
6) Restart your services. To view the configuration service, go to http://localhost:8888/product-service/master.

You will see the following JSON file:
<br>

![image](https://user-images.githubusercontent.com/84160502/177059138-afd10a6c-b039-41e4-a063-e3abd44d30d7.png)

<br>

As the application.properties file contains all the shared properties of the different microservices, we will need other files for the microservice-specific properties. For it:

- Create in the myConfig directory a product-service.properties file for the ProductService service.
- Add your service properties, i.e. for example:
```
me=drissi.houcemeddine@enicarthage.rnu.tn
```

- Restart the configuration microservice. Looking at the url http://localhost:8888/product-service/master, we notice the addition of the new property.
<br>

![image](https://user-images.githubusercontent.com/84160502/177059342-60045356-2a4f-4bda-aec4-b0970d2233e7.png)

<br>

We will now define a REST call to this property. For it:
- Create the ProductRestService class in the product-service project. Its code will look like the following:
```
package tn.enicarthage.overmicro.productservice;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ProductRestService {

    @Value("${me}")
    private String me;


    @RequestMapping("/messages")
    public String tellMe(){
        System.out.println("c'est moi qui a répondu !");
        return me;
    }
}
```
- Restart the three instances of the service, then call the service in your browser by typing: http://localhost:8080/messages. You will see the following result on the browser:

<br>

![image](https://user-images.githubusercontent.com/84160502/180667928-3982b454-0bb0-4e2e-8241-bff74c597ce8.png)

<br>

### MicroserviceDiscoveryService
To avoid a strong coupling between microservices, it is strongly recommended to use a discovery service which makes it possible to save the properties of the different services and thus avoid having to call a service directly. Instead, the discovery service will dynamically provide the necessary information, providing the elasticity and dynamicity inherent in a microservices architecture.

<br>
























