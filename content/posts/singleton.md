---
title: "Design Patterns - Singleton"
date: 2021-04-19T23:49:53+02:00
draft: false
summary: "Singleton is a design pattern that limits the instantiation of a class to a single instance and provides global access to it."
---

Singleton is a design pattern that limits the instantiation of a class to a single instance and provides global access to it.

If we want to have only a single instance of a class and it must be accessible from a well-defined access point we can use singletons. In recent years singletons are seen as [bad practice](https://stackoverflow.com/questions/137975/what-is-so-bad-about-singletons) but they are still one of the most well-known design patterns. They are used in logging, caching, hardware interfaces, and configuration/properties files. Singletons are pretty easy(deceptively) to implement.

For a basic implementation, we need:
* a private constructor to restrict new instance creation
* a static member of singleton type
* a static public getInstance method to give access to this instance

##Basic Implementation
``` java

public class MySingleton {
    private static MySingleton instance = new MySingleton();

    private MySingleton(){}

    public static MySingleton getInstance() {
        return instance;
    }

    public void doStuff() {
        //some work here
    }
}

```

Although this is usable, we generally want to lazily initialize our singleton instance because if we have a lot of singleton classes and if all of them are eagerly loaded, the startup of our application will be affected. 

##Lazy Loading
So we can naively change it to something like this.

``` java

public class MySingleton {
    private static MySingleton instance = null;

    private MySingleton(){}

    public static MySingleton getInstance() {
        if(instance == null) {
            instance = new MySingleton();
        }
        return instance;
    }

    public void doStuff() {
        //some work here
    }
}

```

This is a little bit better but we have new problems now because this implementation is not threadsafe. Also, other people can use reflection to bypass our restriction on the constructor. 

##Thread-Safety
We should make the instance variable _volatile_ to give it visibility across threads and make the instantiation synchronized.

``` java

public class MySingleton {
    private static volatile MySingleton instance = null;

    private MySingleton() {
        if(instance != null) {
            throw new IllegalStateException("Already instantiated");
        }
    }

    public static MySingleton getInstance() {
        if(instance == null) {
            synchronized(MySingleton.class) {
                if(instance == null){
                    instance = new MySingleton();
                }
            }
        }
        return instance;
    }

    public void doStuff() {
        //some work here
    }
}

```

We compare the instance with null twice, the first one is to not block reads to the instance if it is already initialized. If we made the getInstance method synchronized, it would also work but with much worse performance.

##Enum Singletons
Even though it looks like a weird trick, using [enums](https://stackoverflow.com/a/71399) is widely accepted as one of the best ways to implement singletons. We solve the problems with thread-safety, reflection, and lazy loading. I did not write about them here but using Enums solve problems with cloning and serialization as well.

``` java

public enum MySingleton {
    INSTANCE;

    public void doStuff() {
        //some work here
    }
}

```

We should consider these before using singletons:
* They violate the [SRP](https://en.wikipedia.org/wiki/Single_responsibility_principle) since they control their lifecycle and behavior
* They cause the code to be [tightly coupled](https://stackoverflow.com/q/2832017)
* They make testing [harder](https://stackoverflow.com/a/2085988)
* If we are not careful they are not thread-safe

Spring beans have singleton scope by default but they are not exactly the same concept. Java Singletons are scoped by the Java class loader, Spring singletons on the other hand can be any class we have written but spring will only create one instance for that class in that container context. 