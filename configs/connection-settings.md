# Connection settings
- `listen_addresses (string)`
- `port (integer)`
- `max_connections (integer)`

<br>

### listen_addresses (string)
Specifies the TCP/IP address(es) for server's **listening sockets**, the **default** value is `localhost`.<br>

The value takes the form of a **comma-separated list** of **host names** and/or **numeric IP addresses**. 
- the special entry `*` corresponds to **all available IP interfaces**;
- the entry `0.0.0.0` allows listening for **all IPv4** addresses;
- the entry `::` allows listening for **all IPv6** addresses. 
- if the list is **empty**, the server does not listen on any IP interface at all, in which case **only Unix-domain sockets** can be used to connect to it. 

<br>

Examples:
- `listen_addresses='*'`
- `listen_addresses='localhost,{ip}'`

<br>

While client authentication allows fine-grained control over who can access the server, `listen_addresses` controls which interfaces accept connection attempts.

<br>

### port (integer)
The **TCP port** the server listens on, `5432` by **default**.

<br>

# max_connections (integer)
Determines the **maximum** number of **concurrent** connections to the database server, the **default** is typically `100` connections.
