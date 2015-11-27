## Q1
What is a socket?

## Q2 
What is special about listening on port 1000 vs port 2000?

## Q3 
Describe one significant difference between IPv4 and IPv6

## Q4
When and why would you use ntohs?

## Q5
If a host address is 32 bits which IP scheme am I most likely using? 128 bits?

## Q6
Which common network protocol is packet based and may not successfully deliver the data?

## Q7
Which common protocol is stream-based and will resend data if packets are lost?

## Q8
What is the SYN ACK ACK-SYN handshake?

## Q9 
Which one of the following is NOT a feature of TCP?

Packet re-ordering
Flow control
Packet re-tranmission
Simple error detection
Encryption

## Q10
What protocol uses sequence numbers? What is their initial value? And why?

## Q11
What are the minimum network calls are required to build a TCP server? What is their correct order?

## Q12
What are the minimum network calls are required to build a TCP client? What is their correct order?

## Q13
When would you call bind on a TCP client?

## Q14 
What is the purpose of
socket
bind
listen
accept
?

## Q15
Which of the above calls can block, waiting for a new client to connect?

## Q16
What is DNS? What does it do for you? Which of the CS241 network calls will use it for you?

## Q17
For getaddrinfo, how do you specify a server socket?

## Q18
Why may getaddrinfo generate network packets?

## Q19
Which network call specifies the size of the allowed backlog?

## Q20
Which network call returns a new filedescriptor?

## Q21
When are passive sockets used?

## Q22
When is epoll a better choice than select? When is select a better choice than epoll?

## Q23
Will  `write(fd, data, 5000)`  always send 5000 bytes of data? When can it fail?
