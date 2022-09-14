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

This is used when you want to access the Postgresql server from a remote system. Database administration tools such as DBeaver and pgAdmin use the TCP/IP network socket. Given that this is remote access, what we want to do is to minimize the potential risk for anyone attempting to gain access to the system. You implement this when hosting an application on a network. The system should be configured to listen to and accept connections only to required networks. During installation, use the _listen_addresses_ configuration parameter in _postgresql.conf_ to ensure Postgres only listens and accepts connections on the required network addresses, thus preventing access from, say, the storage network.

### Firewall

They are used to prevent access to network ports from unauthorized sources. Most modern operating systems include firewalls. They are used to allow inbound and outbound rules which specify what traffic is allowed. Some of the common parameters used are:

- the protocal (TCP or IPv6)
- the local port (5432, the default for PostgreSQL)
- the source address (where the attempt to connect is coming from)

As always, we want to minimize access to Postgres, so it would be quite normal to create a rule for TCP (and/or IPv6) traffic arriving on port 5432 to be rejected (or black-holed) unless it's coming from the address of our application server. 

Cloud providers usually recommend the use of in-built firewals of their network infrastructure, which generally works the same way as local firewalls.

#### Transport Encryption

If traffic to the database server is flowing across the network, it is good practice (arguably essential practice) to encrypt that traffic. Postgres uses OpenSSL to provide transport security. To encrypt connections in Postgres you will need at least a server certificate and key, ideally protected with a passphrase that can be securely entered at server startup either manually or using a script that can retrieve the passphrase on behalf of the server, as specified using the _ssl_passphrase_command_ configuration parameter.

If you have an existing Certification Authority (CA) in use you can use certificates provided from that with Postgres. The configuration parameters ssl_ca_file and ssl_crl_file allow you to provide the CA (and intermediate) certificates and the certificate revocation list to the server. This gives you the flexibility to revoke certificates in response to security incidents and have the server reject client certificates or the client reject server certificates. It also allows you to configure the client and server to reject each other if the identity of either cannot be verified through the chain of trust to prevent as-yet undetected spoofing.

It's important to ensure that your use of TLS is secure as well. There are a number of configuration parameters that can be set to ensure that you're not using ciphers or other options that may no longer be considered secure. It is recommended that you check and appropriately configure the following configuration parameters in your postgresql.conf configuration file:

- ssl_ciphers
- ssl_ecdh_curve
- ssl_dh_params_file
- ssl_min_protocol_version

## Authentication




## Roles


## Data Access Control


## Encryption

