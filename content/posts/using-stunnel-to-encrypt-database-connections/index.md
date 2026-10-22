---
title: "Using stunnel to Encrypt Database Connections"
date: 2017-07-09
categories: 
  - "recipes"
tags: 
  - "linux"
  - "security"
---

Most RDBMS such as PostgreSQL and MySQL support TLS connections to encrypt the data on the wire between the application and the database. However there also seems to be applications here and there that don't support TLS connections to the database server. Not the end of the world when your app is on the same server as the database server, but who does that? ;-)<!--more-->

That's where [stunnel](https://www.stunnel.org/index.html) comes in to play. `stunnel` is open source software that creates an encrypted tunnel that is very easy to install (most major OS distros have it in their package manager) and there are even binaries available for Android and Windows! But enough about that, as there is plenty of information on the [stunnel site](https://www.stunnel.org/index.html). So how does stunnel work? Here is a simple diagram that shows an overview of stunnel in action:

![stunnel overview](images/stunnel-overview.png)

1. On the database server, PostgreSQL is listening on its standard port on 127.0.0.1:5432 (localhost) and stunnel will listen on an external interface, eg. 192.168.12.34 and on a different port of your choosing (we will use 6543 in this example).
2. Through the configuration of stunnel, traffic coming in to the interface 192.168.12.34:6543 will then pass to PostgreSQL listening on 127.0.0.1:5432.
3. On the application server, stunnel is listening on 127.0.0.1:5432, so the application can be configured to talk to 127.0.0.1:5432, just as if PostgreSQL was installed on the application server instead of across the network.

### Stunnel Configuration

The configuration file format is pretty much the same on the different OSes, you just need to find where your favorite distro stores the configuration file(s) by default. Another benefit of stunnel is that you can run multiple instances for different applications, etc. as long as they are listening on different ports, of course. You can also "multiplex" the connections to one endpoint. For example, you could have multiple application servers using stunnel to connect to the same ip address / port on the database server. So, on to the configuration examples.

We need to setup the keys/certificates for the TLS encryption. For stunnel, there are two ways to do this:

1. PKS (shared keys) - This is the simplest way to setup stunnel and recommended if you don't have an in-house certificate authority or do not want to setup one.
2. PKI (public key) - If you have your own CA setup you can use the CA root certificate and client key/certificate pair.  stunnel can use certificates from a commercial CA as well.

See the [stunnel authentication page](https://www.stunnel.org/auth.html) for more information on these methods.

**PSK Example:**

Say we are are running PowerDNS for our DNS needs and we are using PowerDNS's pgsql backend.  For this example, we will use PKS. On the database server, we will use this stunnel config:

```
; setup stunnel for PostgreSQL server
fips = no

; create a dedicated user/group (optional) 
setuid = stunnel
setgid = stunnel

; make sure the directory has the necessary perms for stunnel to create the pid file
pid = /var/run/stunnel/stunnel.pid

; Log level. WARN=4, DEBUG=7
debug = 4

[pgsql]
client = no
accept = 192.168.12.34:6543
connect = 127.0.0.1:5432
sslVersion = TLSv1.2
ciphers = PSK
PSKsecrets = /etc/stunnel/psk.txt
```

The line "**client = no"** indicates that this is an stunnel server. The PSKsecrets file contains a secret for each client that will connect:

```
app01:GyU6K4d7z7l68OdJ1yYzzltFpnd7wTxWNw8cshYiDd9OZfShmy9nR6bUEfmyjLR
app02:UqvhC7LegFjDDYGWl9LFmh6M7ccNs6v7NEQCQzOyfkY50jNcWriscpDAvXzkk8a
```

Each client connecting to an stunnel server should have its own PSK. Here is the stunnel configuration for the client, in this case app01:

```
; setup stunnel for PowerDNS
fips = no

setuid = stunnel
setgid = stunnel
pid = /var/run/stunnel_pgsql.pid

; Log level. WARN=4, DEBUG=7
debug = 4

[pgsql]
client = yes
accept = 127.0.0.1:5432
connect = 192.168.12.34:6543
sslVersion = TLSv1.2
ciphers = PSK
PSKsecrets = /etc/stunnel/app01_psk.txt
```

Here we set the line "**client = yes**" to indicate that this is an stunnel client. The PSKsecrets file will contain just one line, the key from the app01 line on the server's PSKsecrets file:

```
app01:GyU6K4d7z7l68OdJ1yYzzltFpnd7wTxWNw8cshYiDd9OZfShmy9nR6bUEfmyjLR
```

**PKI Example**

For the PKI example, we use most of the same config, with the exception that we configure the server and client to use certificates and private keys. The setup is very similar to how you would setup an HTTP server to serve TLS. For this we will use an in-house certificate authority. Here is the server configuration:

```
; setup stunnel for PostgreSQL server

fips = no

; create a dedicated user/group (optional)
setuid = stunnel
setgid = stunnel

; make sure the directory has the necessary perms for stunnel to create the pid file
pid = /var/run/stunnel/stunnel.pid

; Log level. WARN=4, DEBUG=7
debug = 4

[pgsql]
client = no
accept = 192.168.12.34:6543
connect = 127.0.0.1:5432
sslVersion = TLSv1.2

; the db_server.pem contains the private key, server certificate, and the CA root certificate
cert = /etc/ssl/private/db_server.pem
```

The .pem file contains the entire chain, e.g.:

```
-----BEGIN PRIVATE KEY-----
MI3f8AM0nsdanraZxJw4T8XMuhdnaDIpLgUaYylqC77533wz2HasAwzeL2HoRekn
 ... private key data
-----END PRIVATE KEY-----
-----BEGIN CERTIFICATE-----
Md3i1dJf8IXfhod7bOzfanBuVlQVrGzEMxvqTXnIpInM8NKlRrLs5T59Zlm8vYUl
 ... server certificate data
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
VQE9qMQkeM3tb7WWLOJfqMD9vRo419xGOySX8h7PLPKYSGGm8ufXKQxj6ok3WTPw
 ... CA certificate
-----END CERTIFICATE-----
```

For the client, the configuration would be:

```
; setup stunnel for PostgreSQL
fips = no
setuid = www-data
setgid = www-data
pid = /var/run/stunnel_pgsql.pid

; Log level. WARN=4, DEBUG=7
debug = 4

[mysql]
client = yes
accept = 127.0.0.1:5432
connect = 192.168.12.34:6543
sslVersion = TLSv1.2

; the client only needs the CA root certificate
CAfile = /etc/ssl/certs/CA_Root.crt
```

While maintaining a certificate authority is not the most fun, it has the advantage that there are no secrets necessary on the client(s), which can also mean it is easier for configuration management tools (Ansible, Puppet, etc.).
