# Welcome to the Merideum Docs!

## Basics
Merideum is an http request protocol and execution engine that aims to make web services more dynamic and code-like.  Merideum includes a programming language called Merit which is used to define a request to the server. The server executes the code and returns a result.

Merit code is written within the request body:

```
request helloWorld {
    const name = "Earth"
    
    const message = "Hello ${name}!"
    
    return message
}
```

When a `POST` is made to `/merideum` with that request body, the result is returned on the response in JSON:

```json
{
    "output": {
        "message": "Hello Earth!"
    }
}
```

### Expanding with Resources
A Merideum server is a repository for resources which can be used within a request. Resources are defined in the server and exposed on the request using the Merideum API.

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
request helloWorld {

    import greeter: Greeter

    const name = "Earth"
    
    return {
        message = greeter.sayHello(name)
    }
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

Since resources are abstract within the request, their implementation in the server is entirely flexible and language specific. A resource may be a package in Go, a collection of exported functions in JavaScript, or even calls to a GraphQL service. Merideum services could be unified under one gateway, or depended upon individually during configuration.

```kotlin
// Using the Ktor plugin
install(Merideum) {
    resources {
        from("http://my-other-service.com/")
    
        resource(Greeter())
    }
}
```

### The Power of Contracts
Contracts are a way to save requests on the server. Contracts can have parameters and return output.

#### Turning the previous examples into a contract

```


contract helloWorld(name: String) {

    import greeter: Greeter
    
    return {
        message = greeter.sayHello(name)
    }
}
```

Request:
Route: `/merideum/contracts/ace3F88jEW2`
```json
{
    "parameters": {
        "name": "Earth"
    }
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

Contracts are a powerful way to extract business logic and code that is duplicated across clients. Since contracts are immutable and trustable, a service can offload logic like input validation and entity mutation to a contract. Contracts also track code steps making them excellent for service monetization and sharing data between parties.

### Expanded Example

The following is an example of a contract that takes input, validates it against a set of rules, transforms the input, saves it to a repository, and returns the id of the entity that was created.

```
contract createPerson(firstName: String, surName: String, age: Int, zipCode: String) {

    import {
        validations: PersonConstants
        locationService: LocationService
        personRepository: PersonRepository
    }
    
    if (validations.validateFirstName(firstName) == false) {
        request.errors["firstNameInvalid"] = "firstName value is invalid."
    }
    
    if (validations.validateLastName(lastName) == false) {
        request.errors["lastNameInvalid"] = "lastName value is invalid."
    }
    
    if (validations.validateAge(age) == false) {
        request.errors["ageInvalid"] = "Age value is invalid."
    }
    
    if (locationService.validateZipCode(zipCode) == false) {
        request.errors["zipCodeInvalid"] = "ZipCode does not exist."
    }
    
    // If there are any errors, do not continue the contract.
    if (request.errors.isNotEmpty()) {
        request.fail()
    }
    
    const state = locationService.stateByZipCode(zipCode)
    
    const person = {
        name = "${firstName} ${surName}",
        age = age,
        state = state
    }
    
    const created = personRepository.create(person)
    
    return created{ id }
}
```
