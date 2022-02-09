# Understanding HTTP Life Cycle

You probably are curious to know what happens when you type in a query in your browser's URL address bar and hit _Enter_. Similarly, you may have just finished deploying your application on a server and you may be curious to understand how your application interacts with the server. In this article, I will break down the entire HTTP operation so that you can understand how it works. This operation is called the _HTTP Life Cycle_.

## What We Will Look At

1. [Composition of a URL](#composition-of-a-url)
2. [The HTTP protocol](#the-http-protocol)
3. [Status codes and their meaning](#status-line)

I will use my personal blog as an example to explain how the HTTP cycle works. To access my blog, I can type the URL _https://www.gitauharrison.com/blog_ in my browser's address bar.

![Personal blog in address bar](/images/http_life_cycle/url_bar.png)

When I hit enter, this is what I get:

![Personal blog](/images/http_life_cycle/personal_blog.png)

So, what happened? From the above URL, `https` is the protocol, `www.gitauharrison.com` is the host (domain name) and `/blog` is the resource path. Domains are mapped to the IP address of the host. Rather than remembering an IP address such as 142.250.203.206, it is much easier to remember a name such as _google.com_. Read more on host names [here](#host-names).

Since the browser now knows what host it is trying to reach, which is www.gitauharrison.com, it will try to connect to the server. The steps involved in resolving the Domain Name System (DNS) include:

- First, the browser checks its local cache to see if the host mapping already exists. If it does, then it will respond by returning the content of the cached host mapping.

- If the host mapping does not exist in the browser cache, DNS resolver checks the Operating system level cache. An operating system keeps its own DNS resolution for websites that were recently surfed by different clients on that machine.

- If the record does not exist in the OS cache, then the Internet Service Provider (ISP) cache is checked. An ISP cache is a cache that is maintained by the ISP. 

- If the record does not exist in the ISP level, then a query is made to the DNS server over the internet for that particular resource.

## Composition of a URL

A URI (Uniform Resource Identifier) is a string of characters that identifies a resource on the Internet. A URL (Uniform Resource Locater) is a type of URI that identifies a resource based on its location rather than its name or attribute. The URL is composed of the following parts:

- **Scheme**: It identifies the protocol used to access a resource on the internet. This can be [HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) or [HTTPS](https://en.wikipedia.org/wiki/HTTPS).
- **Host**: A host name identifies a host that holds a resource, for example, www.gitauharrison.com.
- **Path**: A path identifies a specific resource on the host. For example, _/blog_.
- **Query String**: A query string is a string of characters that is appended to the URL after the path. For example, _?page=1_. This will return only the contents found in page 1 of the blog.
- **Fragment**: A fragment is a string of characters that is appended to the URL after the query string. For example, _#comments_. The blog page might have a section called _comments_. If I want this particular part of the blog page, then the URL would look like https://www.gitauharrison.com/blog?page=1#comments.

The structure of a HTTP URL looks like this:

```html
scheme://host:port/path?query#fragment
```

Hosts can be followed by a port number. The port number is optional. If it is not included, then the default port number is 80. Well known port numbers are normally not included in a URL. Example ports include:

|                 Service              | Port Number |
| ------------------------------------ | ----------- | 
| Hypertext Transfer Protocol (HTTP)   |      80     |
| HTTP with Secure Sockets Layer (SSL) |    443      |
| File Transfer Protocol (FTP)         |      21     |
| Telnet                               |      23     |

## The HTTP Protocol

The correct HTTP fortmat depends on the version fo the HTTP protocol. There are two formarts:

- HTTP/1.0: earlier protocol with fewer functions
- HTTP/1.1: later protocol with more functions

**HTTP request** is normally made by a client to a named host located on the server, with the aim of requesting to access a resource on the server. **HTTP response** is made by the server to a client who made a request, the aim being to provide the requested resource. All these actions are referred to as __requirements__. A client or server that fulfills the requirements for its version of the HTTP protocol is said to be _complaint_ with the HTTP specification.

### HTTP Requests

It is made by a client to a host wanting to access a resource on the server. The request is made by sending a message to the server. The message is composed of the following parts:

- **[Request Line](#request-line)**: The first line of the message. It contains the method, the resource path and the HTTP version.
- **[Header Fields](#header-fields)**: The header fields are the key-value pairs that are sent with the request.
- **[Message Body](#message-body)**: The message body is the content of the request.

#### Request Line

It is the first line in a request message. It contains the _method_, the _resource path_ and the _HTTP version_.

- **Method**: The method is the type of request telling the server what it should do. For example, a server could be asked to send the resource to a client. The most common methods are GET, POST, PUT(replace), DELETE, and PATCH(modify).
- **Resource Path**: The resource path is the path of the resource that the server is asked to send. For example, if the resource path is _/blog_, then the server is asked to send the contents of the blog.
- **HTTP Version**: The HTTP version is the version of the HTTP protocol that the client is using. For example, if the HTTP version is _HTTP/1.1_, then the client is using the latest version of the HTTP protocol.


An example of a request line is:


```html
GET /blog?page=1 HTTP/1.1
```

A request line my contain the _scheme_, _host_ and a _query string_ components. If so, then it is known as the _absolute URL_. For HTTP/1.1, if the host component of the URL is not included in the request line, it must be included in the header fields of the message.

#### Header Fields


The header fields are the key-value pairs that are sent with the request, providing the recipient with information about the message, the sender, and the way the sender wants to communicate with the recipient. Each HTTP header must have a name and a value. A server can use the header fields to decide how to repond to a client's request.

Example of a header field:


```html
Accept-Language: en-US
If-Modified-Since: Sun, 30 Jan 2022 06:20:00 GMT
```

Here, the end user only wants to read the requested resource in American English and that the document should only be sent if it has been modified since the date specified.

The header fields are separated by a new line. The header fields are sent in the order they are presented in the message. The header fields are terminated by a blank line. 

#### Message Body

This is the actual content of the request. For example, if the request is a GET request, then the message body is most likely to be empty. For example, if the request is a POST request, then the message body is the data contained in the form.


### HTTP Responses

They are made a server to a client. The aim is to provide the client with the requested resource, or inform the client that the request was successful or there was an error processing it. The response is composed of the following parts:


- **[Status Line](#status-line)**: The first line of the message. It contains the status code and the reason phrase.
- **[Header Fields](#header-fields)**: The header fields are the key-value pairs that are sent with the response.
- **[Message Body](#message-body)**: The message body is the content of the response.


#### Status Line


The status line is the first line in a response message. It contains the _status code_ and the _reason phrase_. The status code is three-digit number that indicates the result of the response. The reason phrase is a short human-readable phrase that describes the status.


An example of a status line is:


```html
HTTP/1.1 200 OK
```
The HTTP version used in the response is HTTP/1.1. The status code is 200. The reason phrase is OK. The status codes are classified by a range of numbers.

| Status Code |    Meaning    |
| ----------- | ------------- |
| 100 - 199   | Informational |
| 200 - 299   | Success       |
| 300 - 399   | Redirection   |
| 400 - 499   | Client Error  |
| 500 - 599   | Server Error  |

There are no specifications for codes above 599.

#### Header Fields


The header fields are the key-value pairs that are sent with the response, containing information that a client can use to find out more about the response and about the server that sent it. This information could help the client with displaying the response to a user.

An example of a header field is:


```html
Date: Sun, 30 Jan 2022 06:20:00 GMT
Server: Apache/2.4.18 (Ubuntu)
Content-Type: text/html; charset=utf-8
Content-Length: 5
Connection: close
```

The response above tells the client what time it was sent, the server that sent it, the type of the content, the length of the content, and the connection type.

If the request was unsuccessful, headers can be used to tell the client what the error was and how it can be resolved. An empty line is placed after the header fields to separate it from the message body.


#### Message Body


The message body is the content of the response. The message payload of a succesfully processed request will contain the resource requested by the client. For example, if the client requested the resource at _/blog?page=1_, then the message body will contain the contents of the blog found on page 1. However, if the request was unsuccessful, the message body will contain the error message and some actions a client can take to complete the request successfully.


## Further Reading

### Host Names

A website (also known as a host) can be indentified on the internet by a host name such as www.gitauharrison.com. They are sometimes called _domain names_. These names are mapped to an IP (Internet Protocol) address.

A host name is used by a client to make a HTTP request to a host. The user, the person making the request, can specifiy to use the IP address of the host name but it is more conviniet to use the host name. This is because the host name can be remembered easily.