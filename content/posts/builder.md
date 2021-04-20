---
title: "Design Patterns - Builder"
date: 2021-04-19T23:48:35+02:00
draft: false
tags: ["programming", "java", "design-patterns"]
description: "The builder pattern allows us to write readable and understandable code to build complex objects."
---

The builder pattern allows us to write readable and understandable code to build complex objects. 
It is generally used to handle the construction of objects that contain a lot of parameters. 
They are usually implemented as static inner classes. StringBuilder is a good example that is used a lot.

But first, let's talk about the alternatives first.

## Telescoping constructors
In this pattern, we have multiple constructors to support multiple combinations of parameters to create an instance. 

``` java
public class Movie {

  private String id;
  private String title;
  private String director;
  private String producer;

  Movie(String id) {
    this.id = id;
  }

  Movie(String id, String title) {
    this.id = id;
    this.title = title;
  }

  Movie(String id, String title, String producer) {
    this.id = id;
    this.title = title;
    this.producer = producer;
  }

  Movie(String id, String title, String producer, String director) {
    this.id = id;
    this.title = title;
    this.producer = producer;
    this.director = director;
  }

  public String getId() {
    return id;
  }

  public String getTitle() {
    return title;
  }

  public String getDirector() {
    return director;
  }

  public String getProducer() {
    return producer;
  }
}
```

This is pretty straightforward but not very useful as the number of fields increases. 
We create immutable objects with this approach but there are some problems with this approach.
* We need to write a lot of constructors to support all parameter combinations. 
* What if we wanted to create a movie with a director but without a producer? This becomes more of a problem if parameters have the same type.

## Exposed setters
Another way to deal with this problem is by exposing setters for every field we have. 

``` java
public class Movie {

  private String id;
  private String title;
  private String director;
  private String producer;

  public String getId() {
    return id;
  }

  public String getTitle() {
    return title;
  }

  public String getDirector() {
    return director;
  }

  public String getProducer() {
    return producer;
  }

  public void setId(String id) {
    this.id = id;
  }

  public void setTitle(String title) {
    this.title = title;
  }

  public void setDirector(String director) {
    this.director = director;
  }

  public void setProducer(String producer) {
    this.producer = producer;
  }
}
```

This is probably the most used and simplest pattern since we can easily auto-generate setter and getter methods. 
There are some problems with this approach as well
* Exposing setters break immutability, so it is not a good idea for [various reasons](http://www.yegor256.com/2014/06/09/objects-should-be-immutable.html)
* This class doesn't signify which of these setters must be used to create a valid object. So we can create an instance in an invalid state.

## Builder
A simple implementation for the builder pattern for the movie class would look like this

``` java
public class Movie {

  private String id;
  private String title;
  private String director;
  private String producer;

  private Movie(Builder builder) {
    this.id = builder.id;
    this.title = builder.title;
    this.director = builder.director;
    this.producer = builder.producer;
  }

  public String getId() {
    return id;
  }

  public String getTitle() {
    return title;
  }

  public String getDirector() {
    return director;
  }

  public String getProducer() {
    return producer;
  }

  public static class Builder {

    private String id;
    private String title;
    private String director;
    private String producer;

    public Builder(String id) {
      Validate.notBlank(id);
      this.id = id;
    }

    public Builder setTitle(String title) {
      this.title = title;
      return this;
    }

    public Builder setDirector(String director) {
      this.director = director;
      return this;
    }

    public Builder setProducer(String producer) {
      this.producer = producer;
      return this;
    }

    public Movie build() {
      return new Movie(this);
    }
  }
}
```


* If we want to make sure some fields are set, we can get them in the builder, constructor, or we can throw exceptions in the build method or somewhere else by validating them.
* They are very easy to implement
* They have very few drawbacks
* A class with 4 parameters is not hard to build but when you have 9 parameters and 4 of them must not be null, this becomes much more helpful.
* Even though they are generally used as inner static classes we can add a builder to legacy code pretty easily as a separate class

Using generics makes really efficient constructors even for multiple layers of child-parent hierarchies. Let's have a look at a builder for User and Project which extends CompanyEntity and BaseEntity classes.

### BaseEntity
``` java
public abstract class BaseEntity {

  private String id;
  private long createdAt;
  private long updatedAt;

  BaseEntity(Builder builder) {
    Validate.notBlank(id);
    this.id = builder.id;
    this.createdAt = builder.createdAt;
    this.updatedAt = builder.updatedAt;
  }

  public String getId() {
    return id;
  }

  public long getCreatedAt() {
    return createdAt;
  }

  public long getUpdatedAt() {
    return updatedAt;
  }

  protected static abstract class Builder<T extends BaseEntity, B extends Builder> {

    private String id;
    private long createdAt;
    private long updatedAt;

    public B setId(String id) {
      this.id = id;
      return (B) this;
    }

    public B setCreatedAt(long createdAt) {
      this.createdAt = createdAt;
      return (B) this;
    }

    public B setUpdatedAt(long updatedAt) {
      this.updatedAt = updatedAt;
      return (B) this;
    }

    public abstract T build();
  }
}
```

### CompanyEntity
``` java
public abstract class CompanyEntity extends BaseEntity {

  private String companyId;
  private String companyName;

  CompanyEntity(Builder builder) {
    super(builder);
    Validate.notBlank(companyId);
    this.companyId = builder.companyId;
    this.companyName = builder.companyName;
  }

  public String getCompanyId() {
    return companyId;
  }

  public String getCompanyName() {
    return companyName;
  }

  protected static abstract class Builder<T extends CompanyEntity, B extends CompanyEntity.Builder> extends
      BaseEntity.Builder<T, B> {

    private String companyId;
    private String companyName;

    public B setCompanyId(String companyId) {
      this.companyId = companyId;
      return (B) this;
    }

    public B setCompanyName(String companyName) {
      this.companyName = companyName;
      return (B) this;
    }
  }
}
```

### User
``` java
public class User extends CompanyEntity {

  private String username;
  private String firstName;
  private String lastName;

  User(Builder builder) {
    super(builder);
    Validate.notBlank(username);
    this.username = builder.username;
    this.firstName = builder.firstName;
    this.lastName = builder.lastName;
  }

  public String getUsername() {
    return username;
  }

  public String getFirstName() {
    return firstName;
  }

  public String getLastName() {
    return lastName;
  }

  public static class Builder extends CompanyEntity.Builder<User, User.Builder> {

    private String username;
    private String firstName;
    private String lastName;

    public Builder setUsername(String username) {
      this.username = username;
      return this;
    }

    public Builder setFirstName(String firstName) {
      this.firstName = firstName;
      return this;
    }

    public Builder setLastName(String lastName) {
      this.lastName = lastName;
      return this;
    }

    @Override
    public User build() {
      return new User(this);
    }
  }
}
```
### Project
``` java
public class Project extends CompanyEntity {

  private String name;
  private String projectDepartment;

  Project(Builder builder) {
    super(builder);
    Validate.notBlank(name);
    this.name = builder.name;
    this.projectDepartment = builder.projectDepartment;
  }

  public String getName() {
    return name;
  }

  public String getProjectDepartment() {
    return projectDepartment;
  }

  public static class Builder extends CompanyEntity.Builder<Project, Project.Builder> {

    private String name;
    private String projectDepartment;

    public Builder setName(String name) {
      this.name = name;
      return this;
    }

    public Builder setProjectDepartment(String projectDepartment) {
      this.projectDepartment = projectDepartment;
      return this;
    }

    @Override
    public Project build() {
      return new Project(this);
    }
  }
}
```
We can use our builders just like below

``` java
    User user = new User.Builder()
        .setCompanyId("OpsGenie")
        .setId(UUID.randomUUID().toString())
        .setFirstName("asd")
        .build();

    Project project = new Project.Builder()
        .setCompanyId("OpsGenie")
        .setId(UUID.randomUUID().toString())
        .setName("thundra")
        .build();
```

The reason for using two different generics on our Builder classes are; 
* The first one is used in the build method, and it returns the type of the child class.
* The second one is used as the return value of our setter methods in the builder. 
  Without it, our setId method would return a BaseEntity.Builder and we couldn't set the child class fields after that. 