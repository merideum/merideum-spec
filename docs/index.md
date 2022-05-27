# Welcome to the Merideum Docs!

## Basics
Merideum is an http request protocol and execution engine that aims to make web services more dynamic and codelike. A Merideum server is a repository for resources which can be used within a request to execute code. Merideum includes a programming language called Merit which is used to define a request to the server.

Merit code is written within the request body:

```
request helloWorld {
    const name = "Earth"
    
    output message = "Hello $name!"
}
```

When a `POST` is made to `/merideum` with that request body, the result is returned on the response in JSON:

```json
{
    "output": {
        "name": "Hello Earth!"
    }
}
```

### Expanding with Resources
Resources can be `imported` to a request to extend the request's functionality or execute code on the server. Resources are defined in the server's codebase and exposed on the request using the Merideum API.

The resource, defined in Kotlin:
```kotlin
class Greeter {
    fun sayHello(name: String): String {
        return "Hello $name!"
    }
}
```

The resource can then be "exposed" to the request using the Kotlin Merideum API.

```kotlin
// Using the Ktor plugin
install(Merideum) {
    resources {
        resource(Greeter())
    }
}
```

Now the functions of `Greeter` can be used in the request:

```
import greeter: Greeter

request helloWorld {
    const name = "Earth"
    
    output message = greeter.sayHello(name)
}
```
Response:
```json
{
    "output": {
        "message": "Hello Earth!"
    }
}
```

Since resources are abstract within the request, their implementation in the server is entirely flexible and language specific. A resource may be a package in Go, a collection of exported functions in JavaScript, or even calls to a GraphQL service.

