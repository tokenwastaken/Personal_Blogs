
# What is a socket?
A socket is a means for two processes to communicate over a network or on the same host.

This is called Inter Process Communication (IPC). On Unix two important sockets come to mind, Unix Domain Socket (UDS) and Network Socket.

# Unix Domain Socket
UDS is created for processes running on the same host to communicate. 

UDS is bidirectional so both ends of the socket can do read/write operations. 

Unix Sockets are files so the location of UDS is defined by a folder path instead of IP address and port.
 
 
`
 isken/. . . / . . . .socket
`

This is done so that sockets can edit who can access the socket.

Instead of Network Sockets which need routing to go to other end, UDS knows that it is executing on the same system, therefore no need for routing, making them faster than IP sockets.

This also means we don’t need extra switches or checksums to calculate, or insert headers.

TCP connecting to localhost also acts like UDS but is relatively slower.

UDS socket uses kernel memory.

 [![DZ0-ORLSW4-AQnw-D8.jpg](https://i.postimg.cc/Ssj7pbpk/DZ0-ORLSW4-AQnw-D8.jpg)](https://postimg.cc/bDc2QMJB)

# Socket Types
There are two types of sockets to keep in mind when working with Unix sockets.

## Stream Socket

Behaves like TCP, ensures delivery and delivers a stream of bytes.

## Datagram Socket

Delivery is not guaranteed, behaves like UDP Protocol. Connectionless because datagrams don’t need open connection to send data. A datagram is a discrete chunk of data.

## Raw Socket

In addition to Stream and Datagram sockets, we also have Raw sockets where we choose the rules to include. 

ICMP ping uses raw ports because it doesn’t use ports to access the IP, therefore it can bypass the Transport Layer (Layer 4) using raw socket.

An example of Unix socket processes would be when we want to get containers from Docker. Docker uses Unix socket so it can handle permissions.

- Client ( get /container, HTTP request, write)
- Server ( read, send data, write)
- Client ( read data)

Unix Sockets don’t communicate with each other, processes communicate using the Unix sockets.

The server program creates the socket, binds the address to the socket and then listens to accept connections. 

Client connects to the socket by initiating their end socket connects to the socket API to communicate. 

After this ends the connections are cut.

[![rzab6503.gif](https://i.postimg.cc/DfL44w6R/rzab6503.gif)](https://postimg.cc/HjWkfHft)

# IP/Network Sockets

IP sockets are used for processes at distinct hosts to communicate. Layer 3 comes into play here since we need routing.

{protocol, IP, port}
