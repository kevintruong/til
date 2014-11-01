## When I re-run my server code it doesn't work! Why?

By default, after a socket is closed the port enters a time-out state during which time it cannot be re-used ('bound to a new socket').

This behavior can be disabled by setting the socket option REUSEPORT before bind-ing to a port:

```C
    int optval = 1;
    setsockopt(sock_fd, SOL_SOCKET, SO_REUSEPORT, &optval, sizeof(optval));

    bind(sock_fd, ...)
```

## Can a TCP client bind to a particular port?

Yes! In fact outgoing TCP connections are automatically bound to an unused port on the client. Usually it's unnecessary to explicitly set the port on the client because the system will intelligently find an unusued port on a reasonable interface (e.g. the wireless card, if currently connected by WiFi connection). However it can be useful if you needed to specifically choose a particular ethernet card, or if a firewall only allows outgoing connections from a particular range of port values.

To explicitly bind to an ethernet interface and port, call `bind` before `connect`