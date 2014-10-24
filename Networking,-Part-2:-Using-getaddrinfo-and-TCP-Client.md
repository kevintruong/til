# Under construction!!

## How do I use getaddrinfo to convert the hostname into an IP address?

The function getaddrinfo can convert a human readable domain name (e.g. `www.illinois.edu`) into an IPv4 and IPv6 address. In fact it will return a linked-list of addrinfo structs:
```C
struct addrinfo {
    int              ai_flags;
    int              ai_family;
    int              ai_socktype;
    int              ai_protocol;
    socklen_t        ai_addrlen;
    struct sockaddr *ai_addr;
    char            *ai_canonname;
    struct addrinfo *ai_next;
};
```

It's very easy to use. For example, suppose you wanted to find out the IP address of a webserver at www.bbc.com

```C
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>

struct addrinfo hints, *infoptr;

// Unlike heap & stack variables global variables are automatically zeroed
// So no need to use memset here

int main() {
  int result = getaddrinfo("www.bbc.com", NULL, &hints, &infoptr)
  if (result) {
    fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(result));
    exit(1);
  }
  struct addrinfo *p;
  for(p = infoptr; p != NULL; p = p->ai_next) {
      printf("socktype:%d protocol:%d\n",p->ai_socktype,p->ai_protocol);
  }
  freeaddrinfo(infoptr);
  return 0;
}
```

## How do I connect to a TCP server (e.g. web server?)

There are three basic system calls you need to connect to a remote machine:
```
getaddrinfo -- Determine the remote addresses of a remote host
socket  -- Create a socket
connect  -- Connect to the remote host using the socket and address information
```
The `getaddrinfo` call if successful, creates a linked-list of `addrinfo` structs and sets the given pointer to point to the first one.


The socket call creates an outgoing socket and returns a descriptor (sometimes called a 'file descriptor') that can be used with `read` and `write` etc.In this sense it is the network analog of `open` that opens a file stream - except that we haven't connected the socket to anything yet!

Finally the connect call attempts the connection to the remote machine. We pass the original socket descriptor and also the socket address information which is stored inside the addrinfo structure. There are different kinds of socket address structures (e.g. IPv4 vs IPv6) which can require more memory. So in addition to passing the pointer, the size of the structure is also passed:

```C
// Pull out the socket address info from the addrinfo struct:
connect(sockfd, p->ai_addr, p->ai_addrlen)
```

## How do I free the memory allocated for the linked-list of addrinfo structs?

As part of the clean up code call `freeaddrinfo` on the top-most `addrinfo` struct:
```C
void freeaddrinfo(struct addrinfo *ai);
```

## If getaddrinfo fails can I use `strerror` to print out the error?
No. Error handling with `getaddrinfo` is a little different:
*  The return value _is_ the error code (i.e. don't use `errno`)
* Use `gai_strerror` to get the equivalent short English error text:

```C
int result = getaddrinfo(...);
if(result) { 
   char* mesg = gai_strerror(result); 
   ...
}
```

## Can I request only IPv4 or IPv6 connection? TCP only?
Yes! Use the addrinfo structure that is passed into `getaddrinfo` to define the kind of connection you'd like.

For example, to specify stream-based protocols over IPv6:
```C
struct addrinfo hints;
memset(hints,0,sizeof(hints));

hints.ai_family = AF_INET6; // Only want IPv6 (use AF_INET for IPv4)
hints.ai_socktype = SOCK_STREAM; // Only want stream-based connection
```

## What about code examples that use gethostbyname!?

The old gethostbyname is deprecated; it's the old way convert a host name into an IP address. The port address still needs to be manually set using htons function. It's much easier to write code to support IPv4 AND IPv6 using the newer `getaddrinfo`

## Is it that easy!?
Yes and no. It's easy to create a simple TCP client - however network communications offers many different levels of abstraction and several attributes and options that can be set at each level of abstraction (for example we haven't talked about `setsockopt` which can manipulate options for the socket).
For more information see
[[http://www.beej.us/guide/bgnet/output/html/multipage/getaddrinfoman.html]]

