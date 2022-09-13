# How to Secure PostgreSQL

Data safety is at the center of all concerns a company may have. The impact of a data breach can jeopardise the trust customers have on the company. In this article,you will learn how to examine the security of your PostgreSQL deployment.

## Table of Contents

- [Access](#access)
- [Authentication](#authentication)
- [Roles](#roles)
- [Data Access Control](#data-access-control)
- [Encryption](#encryption)


## Access

The focus here is to understand how the PostgreSQL server is connected to and accessed. It is recommended that the [principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege) be followed during configuration. This principle emphasizes that every module (process, user or program) must be able to access only the information and resources that are necessary for its legitimate purpose. 

These are the things we will be looking at regarding the access to PostgreSQL server:

- Physical access
- Connection
- Firewall
- Transport Encryption


### Physical Access

This is the server storage area. It is the first step towards controlling access to the server. Even though difficult, measures have to be put in place to ensure that only authorised personnel have access to the server rooms.

In the case that a co-location facility is used, ensure that the chosen provider has a strictly enforced security policy appropriately designed to prevent unauthorized access, and in facilities that allow users to enter, that locking racks and cages are available to keep other customers away from your hardware.

It is necessary that both cloud providers as well as co-location facilities have appropriate documentation certifying the level of security they provide.

### Connection

We will look at two ways to connect to the PostgreSQL server: through a [Unix Domain Socket (UDS)](#unix-domain-socket) or a [TCP/IP Socket](#tcpip-socket).

#### Unix Domain Socket

UDS are the default method to connect to a Postgres database on Unix-like platforms. These sockets are only accessible from the machine on which they are present, making them safer from remote attacks. These sockets appear as special files on the file system. Access is subject to the same access controls as other files, even though only write permissions are needed to use the socket. Sockets are always owned by the user that the postgres server is running as.

For more flexibility, Postgres can create multiple sockets, though one is created by default. Configuration is done using the unix socket directories option, each of which directories can have different permissions so as to segregate users or applications and help to apply the principle of least privilege.

#### TCP/IP Socket



## Authentication


## Roles


## Data Access Control


## Encryption

