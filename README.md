# Simple Spring MVC/Spring Boot Application
Let's build the simplest Spring Boot REST application possible, from scratch. It's common to use Spring's Initializr, but we'll do without. Not that there's anything wrong with Initializr. But we don't really need all the options it provides right now. We're shooting for the simplest thing possible, after all.

You can use these steps to start a new REST application or add RESTful endpoints to an existing project.

### Step 1: Start with a Maven project
Maven's great. Can you build a Spring Boot project with something else? Absolutely, but for now, we're going to use Maven.

### Step 2: Add the dependencies to pom.xml
Spring Boot uses a Maven <parent> element. The <parent> looks a lot like a <dependency> but is a sibling of <dependencies>. It has a groupId, artifactId, and version. It's special because its version dictates or controls the versions of its children. This ensures that a complicated graph of dependencies is version-compatible. We can't mess things up by inadvertently assigning incompatible versions to our dependencies.
```sh
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <!-- Be sure to change these to match your project! -->
    <groupId>your.groupId</groupId>
    <artifactId>your-artifactId</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>
    
    <!--The Spring Boot starter parent. Controls <dependencies> below.-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.0.RELEASE</version>
        <relativePath />
    </parent>
    
    <dependencies> 
        <!--Spring Boot starter children. No versions needed.-->       
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
        </dependency>
    </dependencies>
        
</project>
```
Of note:
- We use Spring Boot's starter dependencies. The starters bundle or chunk dependencies into groups that work well together. You can include individual dependencies if you need more control, but the starters are a good place to, well, start.
- The spring-boot-devtools aren't strictly necessary. They make development easier by reloading or restarting your app when you make code changes without requiring a manual rebuild and deploy. They also disable some caching behavior so changes to static resources and templates (more on those later) are seen immediately.

Build!
With your dependencies in place, build your project so Maven can fetch missing packages and reference packages in your .m2 directory.

### Step 3: Add a main method
Spring Boot starts Spring MVC with a main method, just like a console application. Unlike our JdbcTemplate project, there's no need to implement CommandLineRunner.

Other Spring MVC application environments have no specific starting method. They're loaded by a web server. One of the genius features of Spring Boot is that the web server is bundled with it! Spring Boot starts the web server instead of vice versa. Hence, the main method.
```sh
package corbos.simplerest;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class App {

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```
A couple of tips:
- Don't nest the class containing your main method "deeper" in your package hierarchy than other components. For example, if you have the following packages:
    ```sh
    organization.app  <-- Your main method goes in a class here.
    organization.app.controllers
    organization.app.data
    ```
    The annotation, @SpringBootApplication, kicks off a component scan starting in the annotated class's package. The scan will miss components shallower in the package hierarchy. As with almost everything in Spring Boot, you can change the behavior, but we don't want to complicate things for our simple REST app.

- If your compiler can't find @SpringBootApplication or SpringApplication, rebuild your project. Maven must not have downloaded and referenced the Spring Boot dependencies.

At this point, if you run your application you'll notice Spring output in your console! Browse to http://localhost:8080 and you should see a generic white label error message. This is a valid Spring Boot application. It just doesn't do much. It's a server that doesn't serve anything.

Use Postman to send a GET request. What is the status? Is there content in the body? How about headers?

### Step 4: @RestController
The Spring MVC framework includes a @RestController annotation to turn any Java class (a POJO) into a web-enabled controller. A @RestController is registered with the Spring DI container just like any other @Component, but it also enables the annotated class to handle RESTful HTTP requests with its methods. Actually, there's remarkably little for the controller to do. By the time it receives the request, the server and Spring MVC have already confirmed the request is well-formed, maps to a URL our controller understands, and uses an HTTP method (GET, POST, DELETE...) that our controller expects. Further, it transformed data in the URL and request body into Java objects and values. It can even validate the data without us having to write a line of code. (More on that later.)

```sh
package corbos.simplerest.controllers;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class SimpleController {

    @GetMapping
    public String[] helloWorld() {
        String[] result = {"Hello", "World", "!"};
        return result;
    }
}
```
That's a lot of new things in 16 lines of code. Let's go over them.

- The first half of the source is filled with imports. GetMapping, RequestMapping, and RestController are Spring MVC types that come along with the spring-boot-starter-web dependency. If your compiler can't find them, rebuild and double check packages and names.
- The annotation, @RestController, notifies Spring MVC that this class should be registered with the Spring application context and that it may contain methods that handle REST requests.
- @RequestMapping("/api") is our first mapping annotation. Mapping (or Routing), determines if a given URL, HTTP method, HTTP header, or media type triggers a specific controller method. By applying this annotation at the class level, we tell Spring MVC that this class can only handle URLs that begin with "/api". We don't have to map requests at the class level, but it's convenient. We can also map requests method by method.
- The method helloWorld returns a String[] to the Spring MVC framework, which then serializes the result to JSON and includes it in the HTTP response body. It might feel like a lot of work to set things up the way Spring wants them, but imagine how much work it would be to do everything ourselves!
- The @GetMapping signals to Spring MVC that this method can only handle HTTP requests using the GET method. It can, but doesn't, further refine the accepted URL.

Now we can truly run our application. Start it up. Navigate to http://localhost:8080/api with a browser or Postman. If all is well, the response should be 200 OK with a JSON body:
```sh
[
    "Hello",
    "World",
    "!"
]
```
That's a JSON array of strings, which is exactly what our helloWorld method returns!

If something is amiss, double check your source and layout.

### Post and Calculate
Our "hello world" method is a little dull. Let's see if we can send data to the server and make it do some work. We'll send three things: two integer operands and one string operator that can be one of four values: +, -, *, or /. We send our values in a POST request. First, add a method to SimpleController.
```sh
@PostMapping("/calculate")
public String calculate(int operand1, String operator, int operand2) {
    int result = 0;
    switch (operator) {
        case "+":
            result = operand1 + operand2;
            break;
        case "-":
            result = operand1 - operand2;
            break;
        case "*":
            result = operand1 * operand2;
            break;
        case "/":
            result = operand1 / operand2;
            break;
        default:
            String message = String.format("operator '%s' is invalid", operator);
            throw new IllegalArgumentException(message);
    }
    return String.format("%s %s %s = %s", operand1, operator, operand2, result);
}
```
Details:
- @PostMapping("/calculate") tells Spring MVC to execute our method if an HTTP request's method is POST and the URL is "/api/calculate". The @PostMapping's relative URL is appended to the @RequestMapping's URL at the top of the class.
- Our method returns a String. Compare that to helloWorld's String[]. @RestController methods are remarkably flexible. Spring MVC will take any type returned and do its best to format it as JSON.
- Our method takes three parameters: int operand1, String operator, and int operand2 (order is the same as a mathematical expression). Parameter names are important. Spring MVC will match parameter names with keys from HTTP request key/values.
- If the request's operator isn't one of the four valid values, our method throws an IllegalArgumentException. Let's see how that affects the HTTP response.

Fire up Postman and send a few POST requests to http://localhost:8080/api/calculate. We send our parameters in the Body as x-www-form-urlencoded key/values.
Mix it up. What happens when we:
- Send a non-numeric operand1? (400 Bad Request)
- Omit operand1? (500 Internal Server Error)
- Send an invalid operator? (500 Internal Server Error, from our IllegalArgumentException)
- Send extra key/values? (200 OK, no problem)

### Delete
We won't go through all of the HTTP methods for our simple application, but we'll do one more. How do we delete something using an id? Our method might be:
```sh
@DeleteMapping("/resource/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void delete(@PathVariable int id) {
    // This is where we would use our id to delete.
}
```
New things:
- @DeleteMapping("/resource/{id}") tells Spring MVC to call our method when the HTTP method is DELETE. Its URL has a named chunk delimited with curly braces. It represents a variable chunk. Its value can be almost anything other than "/" and will match a URL as long as the rest of the URL matches.
- Our method doesn't return a type. It's void. That's fine with Spring MVC. It omits the HTTP response body and returns 200 OK by default.
- @ResponseStatus(HttpStatus.NO_CONTENT) overrides the default and returns a 204 No Content status for every request. If we want something else, this approach would be too rigid.
- @PathVariable tells Spring MVC to find the parameter in the URL. In this case, it's the variable chunk {id}. The parameter's name must match the chunk's name.

Send a few DELETE requests with Postman. Remember to include an integer id in the URL (http://localhost:8080/api/resource/12). What happens if you omit the id or send something non-numeric? What happens if you send the integer 999999999999?
