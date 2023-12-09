---
layout: post
title:  "How to Setup CRUD Webserver Using Java Spark Framework"
---

Java Spark Framework offers you only the essential tools to build a web application. This unopinionated web framework can be a great choice for those who wanted a flexibility and freedom on building their apps. There is no strict rules or practices that is being enforced. You do you.

To be frank with you, Java Spark has a high learning curve. With the minimalistic framework, you also have to deal with some boilerplate code (example: handling CORS, JSON response handling, dll). Other than that, this library is not a popular choice so that there is a smaller pool of people that is ready to answer your question or give a code snippet on Stackoverflow.

Before you continue reading this guide. I need to mention that I don't recommend this framework if you are someone who starts learning Java or web applications in general. If you are this person then Spring Boot, Django, or CodeIgniter3 can be your option because of their great community support.

Still here? good! I hope we can have a good time while we are learning to build a simple web CRUD application with Java Spark.

## Setup project gradle

There are two mainstream way to setup Java project: Gradle or Maven. These are great java build automation tools with their respecticve pros and cons. Personally, I like Gradle's dependency management over Maven because of how it is easy to read. For this guide, we will using Gradle to start our initial project. 

```
gradle init --type java-application
```

Below is the initial project structure from the previous gradle commands. 

```
├── app
│   ├── build.gradle
│   └── src
│       └── main
│           ├── java
│           │   └── cors
│           │       └── App.java
│           └── resources
├── gradle
│   └── wrapper:
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
└── settings.gradle

```

## Add Spark Dependencies
```
dependencies {
    implementation 'com.sparkjava:spark-core:2.9.4'
    implementation 'com.google.code.gson:gson:2.7'

    // add another dependencies to suit your needs below
}
```

## Implement Simple CRUD project

### Retrieve Books
```
public class App {
    public static List<Book> library = new ArrayList<>();
    public static Gson gson = new GsonBuilder().create();

    public static void main(String[] args) {
        get("/books", (req, res) -> library, gson::toJson);

        after("/*", (req, res) -> res.type("application/json") );
    }


    static class Book {
        String id;
        String name;
    }
}
```

### Create Books
```
    public static void main(String[] args) {
        
        ...
        
        post("/books", (req, res) -> {
            String body = req.body();
            Book book = gson.fromJson(body, Book.class);

            books.add(book);
            return books;
        }, gson::toJson);

        ...
    }
```

### Update Book
``` 
    public static void main(String[] args) {
         
        ...
        
        put("/books/:id", (req, res) -> {
            String requestedBookId = req.params("id");
            String body = req.body();
            Book updatedBook = gson.fromJson(body, Book.class);

            List<Book> newLibrary = library.stream()
                    .filter(book -> !requestedBookId.equals(book.id))
                    .collect(Collectors.toList());
            newLibrary.add(updatedBook);
            return library = newLibrary;
        }, gson::toJson);
        
        ...
    }   

```

### Delete Book
```
    public static void main(String[] args) {
         
        ...
        
        delete("/books/:id", (req, res) -> {
            String requestedBookId = req.params("id");

            library = library.stream()
                    .filter(book -> !requestedBookId.equals(book.id))
                    .collect(Collectors.toList());
            return library;
        }, gson::toJson);
        
        ...
    }
```

### Full Snippet of The Project

```
public class App {
    public static List<Book> library = new ArrayList<>();
    public static Gson gson = new GsonBuilder().create();

    public static void main(String[] args) {
        get("/books", (req, res) -> library, gson::toJson);

        post("/books", (req, res) -> {
            String rawRequest = req.body();
            Book newBook = gson.fromJson(rawRequest, Book.class);

            library.add(newBook);
            return library;
        }, gson::toJson);

        put("/books/:id", (req, res) -> {
            String requestedBookId = req.params("id");
            String body = req.body();
            Book updatedBook = gson.fromJson(body, Book.class);

            List<Book> newLibrary = library.stream()
                    .filter(book -> !requestedBookId.equals(book.id))
                    .collect(Collectors.toList());
            newLibrary.add(updatedBook);
            return library = newLibrary;
        }, gson::toJson);

        delete("/books/:id", (req, res) -> {
            String requestedBookId = req.params("id");

            library = library.stream()
                    .filter(book -> !requestedBookId.equals(book.id))
                    .collect(Collectors.toList());
            return library;
        }, gson::toJson);

        after("/*", (req, res) -> res.type("application/json") );
    }


    static class Book {
        String id;
        String name;
    }
}
```

### Run the Project
```
./gradlew build run
```

### Testing Your App

```
curl --location 'http://localhost:4567/books'
```

```
curl --location 'http://localhost:4567/books' \
--header 'Content-Type: application/json' \
--data '{
    "id": 1,
    "name": "new books"
}'
```

```
curl --location --request PUT 'http://localhost:4567/books/1' \
--header 'Content-Type: application/json' \
--data '{
    "id": 1,
    "name": "updated books"
}'
```

```
curl --location --request DELETE 'http://localhost:4567/books/1'
```


