---
title: "Reactive Web Apps with Spring WebFlux"
description: "WebFlux is a web framework based on top of a reactive stack."
tags: ["programming","java"]
date: "2021-06-01T18:00:00+02:00"
draft: false
---

WebFlux is a web framework based on top of a reactive stack.

## Reactive

In this context, *reactive* refers to design guidelines on providing resilient, responsive, scalable and message driven
applications. [Reactive Manifesto](https://www.reactivemanifesto.org) is an important document about this topic.

WebFlux is built on [reactive-streams](https://www.reactive-streams.org/) specification, which only has components like
Subscriber, Publisher, Subscription and Processor. Instead of building their custom implementation based on this
specification, WebFlux uses the [ProjectReactor](https://projectreactor.io/) library. Project Reactor is an incredibly
well implementation of reactive streams with Mono and Flux as their publishers.

## WebMVC and WebFlux

While WebFlux is a new and more modern way of creating web applications, it is not necessarily faster or easier to use.
It is important to note that WebMVC also has a non-blocking io support. However, filters and route functions are not
completely non-blocking.

## Non-blocking

The first things we need to realize with WebFlux is why blocking calls is an undesirable thing. One of the most
important benefits of WebFlux is that it uses a low number of threads compared to WebMVC. Using a low number of threads
can be better because creating each thread has a cost of memory and each context switch wastes some time.

With WebMVC, when we make a database call, the thread can wait until it gets a response because there are usually
hundreds of them to handle concurrent requests.

With WebFlux, we assign a callback function which would work when the database call is finished. The worker thread would
continue consuming tasks from the event-loop. If we design our system correctly, this is a really efficient use of
resources. Although, using a low number of threads mean each one of them is more precious so any blocking call could
result in a huge performance issue.

This is why we should be comfortable working with Publishers and Subscribers in a functional manner.

``` java
public Mono<Record> save(Record record) {
    return recordRepository.save(record)
            .retry(5)
            .timeout(Duration.ofMillis(500))
            .doOnNext(this::sendEvent)
            .doOnError(e -> LOG.error("Something went wrong", e);
}
```

Calling the save method with a Record is not enough to save it to the database. If we were to use project reactor alone,
we would need to subscribe to Publishers manually.

``` java
recordService.saveBatch(records)
    .subscribe(
        i -> System.out.println(i), // on next item
        error -> System.err.println("Error " + error), // on error
        () -> System.out.println("Done") // on complete
    );
``` 

However, we should be returning Mono/Flux in our Controllers anyway. So we don't need to worry about schedulers or
subscriptions and WebFlux should handle subscriptions to our publishers.

``` java
@PostMapping
public Mono<Record> save(@Valid @RequestBody Record record) {
    return recordService.save(record);
}
```

would actually save the item to the database.

There might be some instances where we want to run some functions asynchronously. We should use schedulers in these
cases.

### [Schedulers](https://projectreactor.io/docs/core/release/api/reactor/core/scheduler/Schedulers.html)

> parallel(): Optimized for fast Runnable non-blocking executions

> single(): Optimized for low-latency Runnable one-off executions

> elastic(): Optimized for longer executions, an alternative for blocking tasks where the number of active tasks (and threads) can grow indefinitely

> boundedElastic(): Optimized for longer executions, an alternative for blocking tasks where the number of active tasks (
and threads) is capped

> immediate(): to immediately run submitted Runnable instead of scheduling them (somewhat of a no-op or "null object"
Scheduler)

``` java
recordService.save(record)
    .subscribeOn(Schedulers.boundedElastic())
    .subscribe()
```

### Libraries

Spring already has a strong suite of libraries supporting reactive web apps with non-blocking functions. Some libraries
I had the chance to use are:

* spring security
* spring actuator
* micrometer
* r2dbc
* reactive redis
* spring data
* thymeleaf
* spring cloud sleuth
* webclient

### Spring Cloud Sleuth

Using Thread Local to carry contextual data is easy to accomplish in WebMVC applications because of the
thread-per-request approach.

Carrying contextual data is useful when it is difficult to carry a parameter like an id from WebFilters all the way down
to the layer where we make database calls. For an example, we can use MDC from log4j to append correlationIds to all
logs from a request.

Implementing the same thing in WebFlux is a little more difficult because a low number of threads process callbacks from
different requests concurrently. There is still a way to carry contextual data in WebFlux through subscriber context. We
can implement it ourselves manually, but Sleuth not only makes context propagation easier, it also adds tracing
variables to the context in opentracing format if we want.

### WebClient

WebClient is a replacement for RestTemplate in the reactive world. It is a strong, fast and modern client which can be
used in non-reactive java apps as well. It is really easy to implement retry policies, request timeouts, error handling
and logging after getting used to it.

### WebFilters and WebFlux.fn

WebFilters are the replacement to Filters from WebMVC. They are pretty similar, but they work in a non-blocking way.

We can still use traditional Controllers with annotations like @PostMapping or @GetMapping but there is a functional
alternative in WebFlux named *WebFlux.fn*. HandlerFunction and RouterFunction are interfaces which we can use to create
request handlers.

``` java
HandlerFunction<ServerResponse> helloWorld = 
    request -> ServerResponse.ok().bodyValue("Hello World");
```

``` java
RouterFunction<ServerResponse> route = route()
    .path("/person", builder -> builder
        .GET("/{id}", accept(APPLICATION_JSON), handler::getPerson)
        .GET(accept(APPLICATION_JSON), handler::listPeople)
        .POST("/person", handler::createPerson))
    .build();
```

## Lessons Learned

### Built-in reliability functions

Functions like timeout, retry, log and doOnError are incredibly useful. In fact, not using doOnError might mean missing
errors in some cases.

### BlockHound

BlockHound is a java agent which detects blocking function calls in our Project Reactor apps. It helps by telling us the
line blocking takes place. Since blocking can cause huge performance issues on production, preventing them is a big
issue.

### Publisher and Subscriber Functions

Get comfortable with functions on Mono and Flux. Functions like onErrorResume, onErrorReturn, flatmap, merge or
Mono.defer() may look like optional at first. There are many use cases for them even in the most basic applications.

### Spring MVC dependencies

Make sure you do not have spring-boot-starter-web in your classpath, otherwise your application might not be running in
reactive mode.
From [WebFlux Documents](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-webflux)

> Adding both spring-boot-starter-web and spring-boot-starter-webflux modules in your application results in Spring Boot auto-configuring Spring MVC, not WebFlux. This behavior has been chosen because many Spring developers add spring-boot-starter-webflux to their Spring MVC application to use the reactive WebClient. You can still enforce your choice by setting the chosen application type to SpringApplication.setWebApplicationType(WebApplicationType.REACTIVE).
