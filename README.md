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



























