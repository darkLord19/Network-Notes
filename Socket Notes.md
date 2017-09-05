# U's FTP



### Two types of sockets.
1) Stream Sockets(SOCK_STREAM) - uses TCP internally
2) Datagram Sockets(SOCK_DGRAM) - Connectionless, uses UDP internally


## Stream Sockets:
Stream sockets are reliable two-way connected communication streams.
They are error-free and data arrives sequentially. Have to maintain open connection.
Examples: ssh, in http


## Datagram Sockets:
if we send a datagram, it may arrive. It may arrive out of order. 
If it arrives, the data within the packet will be error-free.
we don't have to maintain an open connection as we do with stream sockets. 
We just build a packet, slap an IP header on it with destination information, and send it out. 
No connection needed.
Examples: DHCP, video/audio streaming


## Byte Order: Little-Endian, Big-Endian
To convert our number in other system's byte order we can use
htons() - host to network short
htonl() - host to network long
ntohs()/ntohl() - network to host short/long

```c
struct sockaddr { 
	unsigned short sa_family;    // address family, AF_INET for IPv4 or AF_INET6 for IPv6 
	char sa_data[14];    // 14 bytes of protocol address i.e. destination address and port number for the socket 
};
```

To deal with sockaddr, there is a structure named sockaddr_in to be used with IPv4
A pointer to a struct sockaddr_in can be cast to a pointer to 
a struct sockaddr and vice-versa. 

```c
struct sockaddr_in { 
	short int sin_family; // Address family, AF_INET 
	unsigned short int sin_port; // Port number 
	struct in_addr sin_addr; // Internet address unsigned 
	char sin_zero[8]; // Same size as struct sockaddr, used for padding
};
```

To access 4-Bytes long IP address:
Let's say we have abc declared as struct sockaddr_in, then
abc.sin_addr.s_addr
will give us 4-Bytes long IP address

Similiar structs exist for IPv6.


## Functions and System Calls explained.

## 1)
```c
int socket(int domain, int type, int protocol) 
```
function  shall  create an unbound socket in a communications domain,
and return a file descriptor that can be used in later function calls that operate on sockets.
It returns socket descriptor or -1 if error occurs.
```
domain = IPv4 or IPv6, 
type = Type of socket i.e stream or datagram,
protocol = TCP or UDP (0 to choose the proper protocol for the given type, or
			we can look up protocol using "getprotobyname()" to look up
			for "tcp" or "udp")
```

## 2)
```c
int bind(int sockfd, struct sockaddr *addr, int addrlength) 
```
It associates socket with a port number on machine.
Then the port number is used by the kernel to match an incoming packet to a certain process's socket descriptor.
```
sockfd is the socket file descriptor returned by socket(). 
my_addr is a pointer to a struct sockaddr that contains information about your address i.e. port and IP address. 
addrlen is the length in bytes of that address.
```

## 3)
```c
int connect(int sockfd, struct sockaddr *serv_addr, int addrlen)
```
function shall attempt to make a connection on a connection-mode socket
```
sockfd is our socket file descriptor, as returned by the socket() call, 
serv_addr is a struct sockaddr containing the destination port and IP address, 
addrlen is the length in bytes of the server address structure.
```

## 4)
```c
int listen(int sockfd, int backlog)
```
What if you don't want to connect to a remote host. 
Say, that you want to wait for incoming connections and handle them in some way. 
The process is two step: first you listen(), then you accept().
```
sockfd is the usual socket file descriptor from the socket() system call. 
backlog: is the number of connections allowed on the incoming queue. 
```
What does that mean? Well, incoming connections are going to wait in this queue until you accept() them 
and this is the limit on how many can queue up. Most systems silently limit this number to about 20;
We need to call bind() before we call listen() so that the server is running on a specific port. 

## 5)
```c
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen)
```
Someone far far away will try to connect() to your machine on a port that you are listen()ing on. 
Their connection will be queued up waiting to be accept()ed. 
You call accept() and you tell it to get the pending connection.
It'll return to you a brand new socket file descriptor to use for this single connection! 
Now we have two file descriptors. The original one is still listening for more new connections, 
and the newly created one is finally ready to send() and recv(). We're
```
sockfd is the listen()ing socket descriptor. 
addr will usually be a pointer to a local struct sockaddr_storage. This is where the information about the incoming connection will go. 
addrlen is a local integer variable that should be set to sizeof(struct sockaddr_storage) before its address is passed to accept(). 
accept() will not put more than that many bytes into addr. If it puts fewer in, it'll change the value of addrlen to reflect that.
```

## 6)
```c
int send(int sockfd, const void *msg, int len, int flags)
```
function  shall  initiate transmission of a message from the specified socket to its peer.
The send() function shall send a message only when the socket is connected. 
If  the  socket is a connectionless-mode socket, the message shall be sent to the pre-specified peer address.
```
sockfd is the socket descriptor you want to send data to (whether it's the one returned by socket() or the one you got with accept().) 
msg is a pointer to the data you want to send, and 
len is the length of that data in bytes. 
We Just set flags to 0 (To more on flags see man send()).
```
send() returns the number of bytes actually sent out.
This might be less than the number you told it to send! 
Remember, if the value returned by send() doesn't match the value in len,
it's up to you to send the rest of the string.

## 7)
```c
int recv(int sockfd, void *buf, int len, int flags)
```
function shall receive a message from a connection-mode or connectionless-mode socket
```
sockfd is the socket descriptor to read from, 
buf is the buffer to read the information into, 
len is the maximum length of the buffer, 
and flags can again be set to 0.
```
recv() returns the number of bytes actually read into the buffer, or -1 on error (with errno set, accordingly.) 
recv() can return 0. This can mean only one thing: the remote side has closed the connection on you! 
A return value of 0 is recv()'s way of letting you know this has occurred.


# Extra
IPv6: 128 bits
IPv4 loopback address: 127.0.0.1 (localhost)
IPv6 loopback address- 0000:0000:0000:0000:0000:0000:0000:0001

### PORT Number: 
16 bit number that is like the local address for the connection(Think IP as hotel and port number as room number!)
Different services on the internet uses well-known different porn numbers.
To see list of them on linux run 
```bash
cat /etc/services
```
PORT numbers under 1024 are special and require OS privilages.

```c
inet_pton()[Presentation to network] converts an IP address to its
binary representation and store in struct in_addr.
inet_ntop()[Network to presentation] converts binary representation from in_addr
to printable numbers and dots notation.
```



