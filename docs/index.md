# Welcome to the Merideum Docs!

Merideum is a web server framework that aims to make web services more dynamic and codelike. A Merideum server is a repository for resources which can be used within a request to execute code. Merideum includes a programming language called Merit which is used to define a request to the server.

## Example

Merit code is written within the request body:

```
request helloWorld {
    const name = "Michael"
    
    output name
}
```

When a `POST` is made to `/merideum` with that request body, the result is returned on the response in JSON:

```json
{
    "output": {
        "name": "Michael"
    }
}
```
