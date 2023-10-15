# Application Programming Interfaces (APIs) In Flask

A simple application such as [this chat app](https://github.com/GitauHarrison/api_in_flask) allows logged-in users to make posts on the home page. This application was specifically built for a web browser (the client). An **HTTP client** is a program or device that initiates a request to a server for a specific resource or information. But, what about other types of clients, say a mobile device running Android or iOS? Should I want to extend this application to cater to mobile devices, for instance, I'll have to either (1) build a web view component of the application to fill the entire screen (which offers very little benefit over accessing it via the device's web browser) or (2) laboriously build a native app. Either way, we do not want to serve HTML responses.

For your reference, these are the topics in our discussion:

- [Overview](00_overview.md) (this article)
- [API Blueprint](01_api_blueprint.md)
- [Resource Representation](02_resource_representation.md)
- [Error Handling](03_error_handling.md)
- [Unique Resource Identifiers](04_unique_resource_identifiers.md)
- [API Authentication](05_api_authentication.md)
- [API-friendly Error Messages](06_api_friendly_error_messages.md)
- [API Testing Using Postman](07_api_testing_postman.md)

### Table Of Content

This article is broken down into the following subsections:

- [Overview](#overview)
- [REST Architecture](#rest-architecture)
    - [Client - Server](#client---server)
    - [Layered System](#layered-system)
    - [Cache](#cache)
    - [Code On Demand (Optional)](#code-on-demand-optional)
    - [Stateless](#stateless)
    - [Uniform Interface](#uniform-interface)
        - [Unique Resource Identifiers](#unique-resource-indentifiers)
        - [Resource Representations](#resource-representations)
        - [Self-descriptive Messages](#self-descriptive-messages)
        - [Hypermedia](#hypermedia)



## Overview

APIs act as low-level entry points into the application. The application's view functions do not return HTML typically consumed by web browsers. Instead, we can work directly with the application's resources, and leave the decision of how to present the information to a user entirely to the client. Such an API can retrieve user and post data, and also allow a user to edit their data or a post without mixing the logic with HTML.

## REST Architecture

What is a REST API? First, REST stands for **Representational State Transfer**. REST is an architecture created to guide the design and development of software. It has six fairly generic and abstract characteristics. The remainder of this article is dedicated to understanding the architectural constraints of REST.

### Client - Server

This principle states that the roles of the client and the server should be clearly differentiated by separating the user interface concerns from the data storage concerns.


### Layered System

This constraint states that a client cannot tell whether it is connected to the end server directly or through an intermediary. If a proxy server or a load balancer is placed between a client and a server, the client should not be able to tell the difference. The principle applies to a server as well. A server may receive requests from an intermediary and not a client directly, so it should not assume that the other side of the connection is a client.

### Cache

An extension of the layered system, this principle allows for a server or an intermediary to cache responses to requests that are received often to improve system performance. Responses must implicitly or explicitly define themselves as cacheable or non-cacheable through **cache controls** to prevent the client from providing stale or inappropriate data in response to further requests. Properly managed caching can partially or even completely eliminate some client-server interactions. In the context of APIs deployed to production, encryption must be used.

### Code On Demand (Optional)

This is optional. It states that a server can provide executable code in response to a client. There needs to be an agreement between the server and the client on what kind of executable code the client can run (could be Java or JavaScript code).

### Stateless

This principle states that a REST API  should not save any state of the client to be recalled every time a given client sends a request. Common web mechanisms such as "remember me" from users navigating pages of the application cannot be used. In a stateless API, every request needs to include all the information that the server needs to identify and authenticate a client and carry out the request. Similarly, the server cannot store any data related to the client connection in a database or any other storage form.

### Uniform Interface
The most important of them all, this principle has four distinguishing aspects:
- [Unique resource identifiers](#unique-resource-identifiers)
- [Resource representations](#resource-representations)
- [Self-descriptive messages](#self-descriptive-messages)
- [Hypermedia](#hypermedia)

#### Unique Resource Identifiers

These are your typical URLs assigned to each resource. For example, the URL associated with a given user can be **_/api/users/<user_id>_** where **_<user_id>_** is the primary key from the database, the identifier, assigned to a user.


#### Resource Representations

When the server and the client exchange information about a resource, they should agree on a format. Most modern APIs use [JSON](https://en.wikipedia.org/wiki/JSON) to build resource representations. An API can choose to support multiple resource representation format by providing the [content negotiation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation) options by which the client and the server can agree on a format they both like.


#### Self-descriptive Messages

Basically, when the server and the client exchange information about a resource, they must include all the information that the other party needs. For example, A HTTP method such as GET indicates that the client wants to retrieve information about a resource, the POST request indicates that the client wants to create a new resource, the PUT or PATCH requests define modifications to an existing resource while DELETE indicates that the client wants to remove a resource. Learn more about the [HTTP Life Cycle](/http_life_cycle.md).


#### Hypermedia

Since the resources in an application are interrelated, this requirement asks that the relationship be included in the resource representations so that the client can discover new resources by traversing relationships (just like you would on a new web page by clicking on links that take you elsewhere). The idea here is to help a client who enters an API discover more resources and learn them by simply following hypermedia links. 

As easy to understand as that is, what complicates this principle's implementation is that the JSON format commonly used for resource representations does not define a standard way to include links, unlike HTML or XML. You'd be forced to use a custom structure or a JSON-proposed extension such as [JSON-API](https://jsonapi.org/) that tries to address this gap.
