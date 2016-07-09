# Effective Java EE tutorial and examples

This project contains a fully working application based on  Java EE 7. Mostly based on Effective Java EE course by Adam Bien at: 

[https://vimeo.com/ondemand/effectivejavaee]()

-----

## Project Folder `doit`

### video 01.Initial Maven Setup

The best way to work with Java EE is to start a Maven project based on an Archetype created by Adam Bien:

> Set up a basic JavaEE ready application with Maven

```sh
mvn archetype:generate -Dfilter=com.airhacks:javaee7-essentials-archetype
```

[https://github.com/AdamBien/javaee7-essentials-archetype]()

Choose version 1.3 of the template, that includes file for JAX-RS Configuration (REST web services)

### 02.Structuring Java EE Applications with BCE

The application will be developed with BCE pattern and module definition will reflect the choice.

More information about the pattern can be found on Google and at:

[https://www.youtube.com/watch?v=grJC6RFiB58]()

For package definition we can have the following convention:

`<company name>.<application name>.<layer>.<component>.<type>`

Because in out case we want to separate Presentation from Business Logic and Integration classes, in our case the `layer` part of the package path can be one of the following:

- presentation (layer)
- business (layer)
- integration (layer)

Our application can contain many components that have connections between each other, and every component is a Java Package that comprehend:

o--| B | C | E |

So, after the component part of the package, we should use one of the following to specify the component part type:

- Boundary
- Control
- Entity

**boundary** are all the classes that have all the interfaces for humans or other objects.

**control** this optional package can contain classes with some logic like validation, complex tasks that cannot stay in the boundary package, etc.

**entity** contains all the classes for persistence of entities

### 03.Coding The BCE Structure

In our "DoIt" application, let's suppose we want to create a new component to manage the Reminders and we want to expose REST web services, we can define the  package as follows:

#### `Package`

- company name: `io.github.dinolupo`
- application name: `doit`
- layer name:	 `business`
- component name: `reminders`
- component layer type name: `boundary`

#### `Class`

- class:				`TodosResource.java`

To begin we add a very simple rest service to the class:

```java
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

@Path("todos")
public class TodosResource {

    @GET
    //@Produces(MediaType.APPLICATION_JSON)
    public String hello(){
        return "Hello, time is " + System.currentTimeMillis();
    }

}
```

### 04.Saving Time With Testing

Let's create a new project that is useful to test the main project.

We call it **doit-st** where st stands for System Test.

Create a simple Maven project with the following pom.xml:

> pom.xml that contains JAX-RS dependencies to test the rest services, including JUnit and Matchers

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>io.github.dinolupo</groupId>
    <artifactId>doit-st</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <dependencies>

        <dependency>
            <groupId>org.glassfish.jersey.core</groupId>
            <artifactId>jersey-client</artifactId>
            <version>2.23.1</version>
        </dependency>
        <dependency>
            <groupId>org.glassfish</groupId>
            <artifactId>javax.json</artifactId>
            <version>1.0.4</version>
        </dependency>
        <dependency>
            <groupId>org.glassfish.jersey.media</groupId>
            <artifactId>jersey-media-json-processing</artifactId>
            <version>2.23.1</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
            <version>1.3</version>
        </dependency>

    </dependencies>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

</project>```

The test class is the following:

```java
import static org.hamcrest.CoreMatchers.*;
import static org.junit.Assert.assertThat;

import org.junit.Before;
import org.junit.Test;

import javax.ws.rs.client.Client;
import javax.ws.rs.client.ClientBuilder;
import javax.ws.rs.client.WebTarget;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;

public class TodosResourceTest {

    private Client client;
    private WebTarget target;

    @Before
    public void setUp() throws Exception {
        client = ClientBuilder.newClient();
        target = client.target("http://localhost:8080/doit/api/todos");
    }

    @Test
    public void fetchTodos() throws Exception {
        Response response = target.request(MediaType.TEXT_PLAIN).get();
        assertThat(response.getStatus(),is(200));
        String payload = response.readEntity(String.class);
        assertThat(payload, startsWith("Hello, time is"));
    }
}
```

Later on this can become a build job in a Jenkins pipeline to do continuous integration / deployment.


### 06.The Entity In BCE

Instead of a String we should manage entities, so we create an Entity class:


```java
package io.github.dinolupo.doit.business.reminders.entity;

import javax.xml.bind.annotation.XmlAccessType;
import javax.xml.bind.annotation.XmlAccessorType;
import javax.xml.bind.annotation.XmlRootElement;

@XmlRootElement
@XmlAccessorType(XmlAccessType.FIELD)
public class ToDo {
    private String caption;
    private String description;
    private int priority;

    public ToDo(String caption, String description, int priority) {
        this.caption = caption;
        this.description = description;
        this.priority = priority;
    }

    public ToDo() {
    }
}
```

modify the rest service as follows:

```java 
    @GET
    public ToDo hello(){
        return new ToDo("Implement Rest Service", "modify the test accordingly", 100);
    }
```

and the test accordingly:

```java
    @Test
    public void fetchTodos() throws Exception {
        Response response = target.request(MediaType.APPLICATION_JSON).get();
        assertThat(response.getStatus(),is(200));
        JsonObject payload = response.readEntity(JsonObject.class);
        assertThat(payload.getString("caption"), startsWith("Implement Rest Service"));
    }
```

### 07.Create Read Update Delete With JAX-RS

In this step we create simple CRUD services and modify the test class accordingly.

> modified methods of ToDoResource.java

```java
    @GET
    @Path("{id}")
    public ToDo find(@PathParam("id") long id){
        return new ToDo("Implement Rest Service Endpoint id="+id, "modify the test accordingly", 100);
    }

    @DELETE
    @Path("{id}")
    public void delete(@PathParam("id") long id) {
        System.out.printf("Deleted Object with id=%d\n", id);
    }

    @GET
    public List<ToDo> all() {
        List<ToDo> all = new ArrayList<>();
        all.add(find(42));
        return all;
    }

    @POST
    public void save(ToDo todo) {
        System.out.printf("Saved ToDo: %s\n", todo);
    }
```

**we have used POST because later on we will use a technical key in the Entity. If you use a business key, a key with a meaning for the business, then use PUT so you can pass a key along with the client call. PUT is idempotent.**


> test class

```java
  @Test
    public void crud() throws Exception {
        // GET all
        Response response = target.request(MediaType.APPLICATION_JSON).get();
        assertThat(response.getStatusInfo(),is(Response.Status.OK));
        JsonArray allTodos = response.readEntity(JsonArray.class);
        assertFalse(allTodos.isEmpty());

        JsonObject todo = allTodos.getJsonObject(0);
        assertThat(todo.getString("caption"), startsWith("Implement Rest Service"));

        // GET with id
        JsonObject jsonObject = target.
                path("42").
                request(MediaType.APPLICATION_JSON).
                get(JsonObject.class);

        assertTrue(jsonObject.getString("caption").contains("42"));

        Response deleteResponse = target.
                path("42").
                request(MediaType.APPLICATION_JSON)
                .delete();

        // DELETE
        assertThat(deleteResponse.getStatusInfo(),is(Response.Status.NO_CONTENT));

    }
```


### 08.Separating The Concerns of A Boundary

Mantaining all the business behaviours in the Rest resource class, can be complicated when you want to create unit tests. So you shoud put all the behaviours in a separate class and inject an instance into the rest resource:

> move all the behaviours into a EJB manager class:

```java
@Stateless
public class TodosManager {

    public ToDo findById(long id) {
        return new ToDo("Implement Rest Service Endpoint id="+id, "modify the test accordingly", 100);
    }

    public void delete(long id) {
        System.out.printf("Deleted Object with id=%d\n", id);
    }

    public List<ToDo> findAll() {
        List<ToDo> all = new ArrayList<>();
        all.add(findById(42));
        return all;
    }

    public void save(ToDo todo) {
        System.out.printf("Saved ToDo: %s\n", todo);
    }
}
``` 

> Inject the property into the resource class

```java
@Stateless
@Path("todos")
public class TodosResource {

    @Inject
    TodosManager todosManager;

    @GET
    @Path("{id}")
    public ToDo find(@PathParam("id") long id){
        return todosManager.findById(id);
    }

    @DELETE
    @Path("{id}")
    public void delete(@PathParam("id") long id) {
        todosManager.delete(id);
    }

    @GET
    public List<ToDo> all() {
        return todosManager.findAll();
    }
    
    @POST
    public void save(ToDo todo) {
        todosManager.save(todo);
    }
}
```

### 09.Saving The State With JPA

After creating the `TodosManager` class, that is a protocol neutral boundary class, we can use it to save data with JPA. 

1) Annotate the ToDo class with `@Entity`

2) Annotate the `id` property with `@Id` and `@GeneratedValue` since it is a technical key

3) Create a persistence unit, putting the `META-INF/persistance.xml` in the war package:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">
    <persistence-unit name="production" transaction-type="JTA">
        <properties>
            <property name="javax.persistence.schema-generation.database.action" value="drop-and-create"/>
        </properties>
    </persistence-unit>
</persistence>
```

4) Change `TodosManager` to use JPA as follows:

> inject the EntityManager

```java
@Stateless
public class TodosManager {

    @PersistenceContext
    EntityManager entityManager;

    public ToDo findById(long id) {
        return entityManager.find(ToDo.class, id);
    }


    public void delete(long id) {
        ToDo reference = entityManager.getReference(ToDo.class, id);
        entityManager.remove(reference);
    }


    public List<ToDo> findAll() {
        return entityManager.createNamedQuery(ToDo.findAll, ToDo.class).getResultList();
    }

    public void save(ToDo todo) {
        entityManager.merge(todo);
    }
}
```

### 10.Fixing The HTTP POST Implementation

As stated by the HTTP RFC at [https://tools.ietf.org/html/rfc2616#page-54]() that says:

_If a resource has been created on the origin server, the response
   SHOULD be 201 (Created) and contain an entity which describes the
   status of the request and refers to the new resource, and a Location
   header_

we should return 201 and a Location header instead of void (No Content). So let's adjust the POST method as follows:

1) generate getters in the `ToDo` bean

2) change `save` method such that it returns the `ToDo` saved object

```java
@POST
    public Response save(ToDo todo, @Context UriInfo uriInfo) {
        ToDo savedObject = todosManager.save(todo);
        long id = savedObject.getId();
        URI uri = uriInfo.getAbsolutePathBuilder().path("/" + id).build();
        Response response = Response.created(uri).build();
        return response;
    }
```

3) Adjust the test accordingly:

```java
   @Test
    public void crud() throws Exception {

        // create an object with POST
        JsonObjectBuilder jsonObjectBuilder = Json.createObjectBuilder();
        JsonObject todoToCreate = jsonObjectBuilder
                .add("caption", "Implement Rest Service with JPA")
                .add("description", "Connect a JPA Entity Manager")
                .add("priority", 100).build();

        Response postResponse = target.request().post(Entity.json(todoToCreate));
        assertThat(postResponse.getStatusInfo(),is(Response.Status.CREATED));

        String location = postResponse.getHeaderString("Location");
        System.out.printf("location = %s\n", location);

        // GET with id, using the location returned before
        JsonObject jsonObject = client.target(location)
                .request(MediaType.APPLICATION_JSON)
                .get(JsonObject.class);

        assertTrue(jsonObject.getString("caption").contains("Implement Rest Service with JPA"));

        // GET all
        Response response = target.request(MediaType.APPLICATION_JSON).get();
        assertThat(response.getStatusInfo(),is(Response.Status.OK));
        JsonArray allTodos = response.readEntity(JsonArray.class);
        assertFalse(allTodos.isEmpty());

        JsonObject todo = allTodos.getJsonObject(0);
        assertThat(todo.getString("caption"), startsWith("Implement Rest Service"));
        System.out.println(todo);

		 // DELETE
        Response deleteResponse = target.
                path("42").
                request(MediaType.APPLICATION_JSON)
                .delete();

        
        assertThat(deleteResponse.getStatusInfo(),is(Response.Status.NO_CONTENT));
    }
```

4) paying attention to the fact that the delete method could raise an unchecked exception `EntityNotFoundException` when you try to delete a proxied non-existent object, so we adjust like the following:

```java
    public void delete(long id) {
        try {
            ToDo reference = entityManager.getReference(ToDo.class, id);
            entityManager.remove(reference);
        } catch (EntityNotFoundException ex) {
            // we want to remove it, so do not care of the exception
        }
    }
```
 
