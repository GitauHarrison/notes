# Understanding HTTP Life Cycle

You probably are curious to know what happens when you type in a query in your browser's URL address bar and hit _Enter_. Similarly, you may have just finished deploying your application on a server and you may be curious to understand how your application interacts with the server. In this article, I will break down the entire HTTP operation so that you can understand how it works. This operation is called the _HTTP Life Cycle_.

## What We Will Look At

1. [Composition of a URL](#composition-of-a-url)
2. The HTTP protocol
3. Status codes and their meaning

I will use my personal blog as an example to explain how the HTTP cycle works. To access my blog, I can type the URL _https://www.gitauharrison.com/blog_ in my browser's address bar.

![Personal blog in address bar](/images/http_life_cycle/url_bar.png)

When I hit enter, this is what I get:

![Personal blog](/images/http_life_cycle/personal_blog.png)

So, what happened? From the above URL, `https` is the protocal, `www.gitauharrison.com` is the host (domain name) and `/blog` is the resource path. Domains are mapped to the IP address of the host. Rather than remembering an IP address such as 142.250.203.206, it is much easier to remember a name such as _google.com_.

Since the browser now knows what host it is trying to reach, which is _https://www.gitauharrison.com/blog_, it will try to connect to the server. The steps involved in resolving the Domain Name System (DNS) include:

- First, the browser checks its local cache to see if the host mapping already exists. If it does, then it will respond by returning the content of the cached host mapping.

- If the host mapping does not exist in the browser cache, DNS resolver checks the Operating system level cache. An operating system keeps its own DNS resolution for websites that were recently surfed by different clients on that machine.

- If the record does not exist in the OS cache, then the Internet Service Provider (ISP) cache is checked. An ISP cache is a cache that is maintained by the ISP. 

- If the record does not exist in the ISP level, then a query is made to the DNS server over the internet for that particular resource.

## Composition of a URL

A URI (Uniform Resource Identifier) is a string of characters that identifies a resource on the Internet. A URL (Uniform Resource Locater) is a type of URI that identifies a resource based on its location rather than its name or attribute. The URL is composed of the following parts:

- Scheme: It identifies the protocol used to access a resource on the internet. This can be [HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) or [HTTPS](https://en.wikipedia.org/wiki/HTTPS).
- Host: A host name identifies a host that holds a resource, for example, www.gitauharrison.com.
- Path: A path identifies a specific resource on the host. For example, _/blog_.
- Query String: A query string is a string of characters that is appended to the URL after the path. For example, _?page=1_. This will return only the contents found in page 1 of the blog.
- Fragment: A fragment is a string of characters that is appended to the URL after the query string. For example, _#comments_. The blog page might have a section called _comments_. If I want this particular part of the blog page, then the URL would look like _https://www.gitauharrison.com/blog?page=1#comments_.

The structure of a URL looks like this:

```python
scheme://host:port/path?query#fragment
```

Hosts can be followed by a port number. The port number is optional. If it is not included, then the default port number is 80. Well known port numbers are normally not included in a URL. Example ports include:

|                 Service            | Port Number |
| ---------------------------------- | ----------- | 
| Hypertext Transfer Protocol (HTTP) |      80     |
| HTTP with Secure Sockets Layer (SSL) |    443    |
| File Transfer Protocol (FTP)       |      21     |
| Telnet                             |      23     |

