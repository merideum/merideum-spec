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

### The Power of Contracts
Contracts are a way to save requests within the server itself. Contracts can have parameters and output.

#### Turning the previous examples into a contract

```
import greeter: Greeter

contract helloWorld(name: String) {
    output message = greeter.sayHello(name) 
}
```

Example request:
Route: `/merideum/contracts/ace3F88jEW2`
```json
{
    "parameters": {
        "name": "Earth"
    }
}
```

Contracts are a powerful way to extract business logic and code that is duplicated across clients. Since contracts are immutable and trustable, a service can offload logic like input validation and entity mutation to a contract. Contracts also track code steps making them excellent for service monetization and sharing data between parties.

### Expanded Example

The following is an example of a contract that takes input, validates it against a set of rules, transforms the input, and saves it.

```
import validations: PersonConstants
import locationService: LocationService
import personRepository: PersonRepository

contract createPerson(firstName: String, surName: String, age: Int, zipCode: String) {
    
    if (validations.validateFirstName(firstName) == false) {
        error "firstName value is invalid."
    }
    
    if (validations.validateLastName(lastName) == false) {
        error "lastName value is invalid."
    }
    
    if (validations.validateAge(age) == false) {
        error "Age value is invalid."
    }
    
    if (locationService.validateZipCode(zipCode) == false) {
        error "ZipCode does not exist."
    }
    
    // If there are any errors, do not continue the contract.
    if (errors.isNotEmpty()) {
        fail()
    }
    
    const state = locationService.stateByZipCode(zipCode)
    
    const person = {
        name = "${firstName} ${surName}",
        age = age,
        state = state
    }
    
    const created = personRepository.create(person)
    output id = created.id
}
```
