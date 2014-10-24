## How do I use getaddrinfo to lookup the address of a particular machine?

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
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>

struct addrinfo hints, *infoptr;

int main() {
  int result = getaddrinfo("www.bbc.com", NULL, &hints, &infoptr)
  if (result) {
    fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(result));
    exit(1);
  }
  struct addrinfo *p;
  for(p = infoptr; p != NULL; p = p->ai_next) {
     puts(p->ai_canonname);
  }
  return 0;
}

## How do I connect to a TCP server (e.g. web server?)

There are three basic system calls you need to connect to a remote machine:
```C
socket
getaddrinfo
connect
```
The socket call creates an outgoing socket and returns a descriptor (sometimes called a 'file descriptor') that can be used with `read` and `write` etc.In this sense it is the network analog of `open` that opens a file stream - except that we haven't connected the socket to anything yet!

The `getaddrinfo` call if successful creates at least one `addrinfo` struct and sets the given pointer to point to the result. Note the result is actually a linked-list of addrinfo structs!



## How do I free the memory allocated for the linked-list of addrinfo structs

Call `freeaddrinfo` on the top-most `addrinfo` struct.
void freeaddrinfo(struct addrinfo *ai);

## But I've seen code examples that use gethostbyname!?

The old gethostbyname is the old way convert a host name into an IP address. The port address still needs to be manually set using htons function. It's much easier to write code to support IPv4 AND IPv6 using the newer getaddrinfo, which does all of the heavy lifting for you.

More information
http://www.beej.us/guide/bgnet/output/html/multipage/getaddrinfoman.html

