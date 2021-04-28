---
title: "Python Type Hints"
date: 2021-04-27T00:28:43+02:00
draft: false
description: "typing is a standard library module which provides a runtime to support type hints"
tags: ["programming", "python"]
---

The programming language I feel the most proficient and effective in is definitely Java, so I feel more at home with
statically typed languages. Even though I really like Java and [Spring Boot](https://spring.io/projects/spring-boot) for
large-scale web applications, Python is my go-to language for one-time-scripts, smaller apps and
especially [aws lambda functions](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python.html)

I've been writing a lot more python recently, and I really liked a library called typing.
_typing_ is a standard library module which provides a runtime to support type hints.

### Type Hints

_type hinting_ in this context means providing the types for parameters and returned objects.  
This is a simple example that we can see in the [docs](https://docs.python.org/3/library/typing.html)

``` python
def greeting(name: str) -> str:
    return 'Hello ' + name
```

- a parameter _name_ with the type of string
- returned object of type string

Apart from str, we can use types like List, Set, Dict or Tuple.

### Static Type Checking?

It is important to note that Python runtime does not enforce anything regarding these types. They are just "hints"
You might be wondering about the usefulness of this library if there is no enforcement but there are many benefits we
get.

- Dramatically reduces the number of TypeErrors you get.
- Helps IDEs and static analyzers reason about your code. They can analyze it much more accurately with types
- Helps documentation

### Where to Use Them?

In my opinion, If multiple people are working on a python codebase, it can be a great asset to the team because it makes
the code easier understand for everyone involved

It adds a development time overhead so might not be needed in throwaway one-time scripts, although we can still see the
benefit of type safety especially if it is a critical task.

``` python
def evaluate_users(user_ids: List[str]) -> List[Any]:
    user_details = []
    for id in user_ids:
        user_details.append(get_random_user_details(id))

    return user_details


def get_random_user_details(user_id: str) -> Dict[str, Union[str, int]]:
    name = ''.join(random.choices(string.ascii_uppercase + string.digits, k=10))
    age = random.randint(1, 99)
    return {"id": user_id, "name": name, "age": age}

```

If the expected type for user_id was an int instead of str, I would see a warning like this in IDEA,
![IDE warning](/images/typing/ide-warning.png)

Similarly, if the return does not match the expected one, the warning would look like this.
![IDE warning](/images/typing/return-type.png)

There are other benefits of this library, but I think parameter and return types are enough to consider using it
