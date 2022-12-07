# PostgreSQL Database Access Management

Data safety is at the center of all concerns a company may have. The impact of a data breach can jeopardise the trust customers have on the company. In this article,you will learn how to examine the security of your PostgreSQL deployment.

For reference, these are the topics we will be covering in the entire database tutorial:

1. [Postgres Overview](/databases/00_postgresql_overview.md)
2. [Install PostgreSQL](/databases/01_install_postgresql.md)
3. [Access PostgreSQL Using PSQL](/databases/access_postgresql/psql.md)
4. [Access PostgreSQL Using DBeaver](/databases/access_postgresql/dbeaver.md)
5. [How to Secure PostgreSQL](/databases/02_how_to_secure_postgresql.md) (this article)
6. [Getting Started With PostgreSQL](/databases/03_getting_started_with_postgresql.md)
7. [Sample Flask Project Using PostgreSQL](/databases/project_configure_postgres_in_flask.md)


### Table of Contents

This article has the following sub-sections. You can click on any of the links below to navigate to a specific section:

- [Access](#access)
- [Authentication](#authentication)
- [Roles](#roles)


## Access

The focus here is to understand how the PostgreSQL server is connected to and accessed. It is recommended that the [principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege) be followed during configuration. This principle emphasizes that every module (process, user or program) must be able to access only the information and resources that are necessary for its legitimate purpose. 

These are the things we will be looking at regarding the access to PostgreSQL server:

- [Physical access](#physical-access)
- [Connection](#connection)
- [Firewall](#firewall)
- [Transport Encryption](#transport-encryption)


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

### Transport Encryption

If traffic to the database server is flowing across the network, it is good practice (arguably essential practice) to encrypt that traffic. Postgres uses OpenSSL to provide transport security. To encrypt connections in Postgres you will need at least a server certificate and key, ideally protected with a passphrase that can be securely entered at server startup either manually or using a script that can retrieve the passphrase on behalf of the server, as specified using the _ssl_passphrase_command_ configuration parameter.

If you have an existing Certification Authority (CA) in use you can use certificates provided from that with Postgres. The configuration parameters ssl_ca_file and ssl_crl_file allow you to provide the CA (and intermediate) certificates and the certificate revocation list to the server. This gives you the flexibility to revoke certificates in response to security incidents and have the server reject client certificates or the client reject server certificates. It also allows you to configure the client and server to reject each other if the identity of either cannot be verified through the chain of trust to prevent as-yet undetected spoofing.

It's important to ensure that your use of TLS is secure as well. There are a number of configuration parameters that can be set to ensure that you're not using ciphers or other options that may no longer be considered secure. It is recommended that you check and appropriately configure the following configuration parameters in your postgresql.conf configuration file:

- ssl_ciphers
- ssl_ecdh_curve
- ssl_dh_params_file
- ssl_min_protocol_version

## Authentication

This comes after access, how we control users and how can or cannot connect to the server successfully using _pg_hba.conf_ configuration file. This file, found in the postgresql data directory, defines the access rules and authentication methods for the data server. The lines in this file are sequentially processed when a connection is being established. The first line that produces a match is usually used to dertermine the authentication method.

### Example lines

There are multiple formats for the lines in _pg_hba.conf_ file.

```python
local        test_db        sample_user        scram-sha-256
```

The line above will allow a _sample_user_ to connect to _test_db_ over a local UDS using the scram-sha-256.

```python
host        another_db        another_user        172.217.170.174        md5
```

This second line attempts to connect _another_user_ to _another_db_ from the address 172.217.170.174 using the md5 authentication method. The line formart usually is:

- the connection type
- the database name
- the user name
- the client network address/subnet (where needed)
- the authentication method

To ensure that the connection is encrypted, the network connections should use either the _hostssl_ or the _hostgssenc_ connection types. Below are some common authenitication types:

#### Trust

This method should be used with caution. It allows a matching client to connect to the server with no further authentication. This method is especially useful for testing and development on a local machine through UDS connection and where authorised users have access to the machine. In the event there is no other way to reset a server password, it is a useful mechanism. Once granted, the user is to be allowed to reset the password and the privilege revoked or disabled again.

#### Peer and Ident

Both are methods used to allow users to be authenticated by the underlying Operating System. PostgreSQL typically comes pre-configured to use peer authentication. The peer method is only available for local connections. When used, the server gets the user name from the operating system and checks that it matches the requested database user name.

The ident method is available only for network connections. It works similarly to peer authentication, except that it relies on an ident server running on the client to confirm the user name. Both peer and ident allow the use of connection maps to handle acceptable mismatches between the username known to the client and that known to the database server.

#### md5 and scram

The preferred hashing mechanism for use with PostgreSQL has been, and still is, `md5`. It is, however, recommended that users move to the `scram-sha-256` where password authentication is required.

Both md5 and scram-sha-256 use a challenge response mechanism to prevent sniffing, and store hashed passwords on the server, however, scram-sha-256 stores the hashes in what is currently considered to be a cryptographically secure form to avoid issues if an attacker gains access to the hash.

If you need to support password authentication with a standalone Postgres server, you should be using scram-sha-256 as the authentication method. Do not use md5 in new deployments!


## Roles

Roles are used to limit database access for specified users. A role can be a member of other roles, or have other roles that are members of it. This can be referred to as granting a role another role. See the example below:

```sql
GRANT my_super_users TO harry;
```

Here, _harry_ is being made a member of the _my_super_users_ group, which gives him access to the extended functionality reserved for super users and members of the _my_super_users_ role.

### Role attributes

- LOGIN - can this role be used to login to the database server?
- SUPERUSER - is this role a superuser?
- CREATEDB - can this role create databases?
- CREATEROLE - can this role create new roles?
- REPLICATION - can this role initiate streaming replication?
- PASSWORD - the password for the role, if set.
- BYPASSRLS - can this role bypass Row Level Security checks?
- VALID UNTIL - an optional timestamp after which time the password will no longer be valid.

A `SUPERUSER` flagged role is set bypass all checks except the right to login. It is highly recommended to use this role with extreme caution.

