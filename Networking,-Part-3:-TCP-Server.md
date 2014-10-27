## What is `ntohs` and when is it used?

## What are the 'big 4' network calls used to create a server?

The four system calls required to create a TCP server are: `socket`, `bind` `listen` and `accept`. Each has a specific purpose and should be called in the above order

The port information (used by bind) can be set manually (many older IPv4-only C code examples do this), or be created using `getaddrinfo`

We also see examplesof setsockopt later too.

## What is the purpose of calling `socket`?

To create a endpoint for networking communication. A new socket is not particularly useful; though we've specified either a packet or stream-based connections it is not bound to a particular network interface or port. Instead socket returns a network descriptor that can be used with later calls to bind,listen and accept.

## What is the purpose of calling `bind`

The `bind` call associates an abstract socket with an actual network interface and port. It is possible to call bind on a TCP client however it's unusually unnecessary as you don't care about the specific port used.

## What is the purpose of calling `listen`
The `listen` call specifies the queue size for the number of incoming, unhandled connections i.e. that have not yet been assigned a network descriptor by `accept`
Typical values for a high performance server are 128 or more.

## Why are server sockets passive?
Server sockets do not actively try to connect to another host; instead they wait for incoming connections. Aditionally, server sockets are not closed when the peer disconnects. Instead when a remote client connects, it is bumped to different port number for future communications.

## What is the purpose of calling `accept`
Once the server socket has been initialized the server calls `accept` to wait for new connections. Unlike `socket` `bind` and `listen`, this call will block. i.e. if there are no new connections, this call will block until a new client connects.

Note the `accept` call returns a new file descriptor. This file descriptor is specific to a particular client. It is common programming mistake to use the original server socket descriptor for server I/O and then wonder why networking code has failed.


## What are the gotchas of creating a TCP-server?

Using the socket descriptor of the passive server socket (described above)
Not specifying SOCK_STREAM requirement for getaddrinfo
Not being able to re-use an existing port
Not initializing the unused struct entries.

Note, ports are per machine- not per process or per user. In other words,  you cannot use port 1234 while another user is using that port.

## Server code example