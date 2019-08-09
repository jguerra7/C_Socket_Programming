
# Socket Programming 101

# Why Socket Programming ?
    “To build any Networked Application”
- WWW ( Internet Explorer , Firefox )
- FTP (WS FTP )
- P2P (Limewire, Bitcomet)

## Pre - Requisites
```
Basics of IP / TCP / UDP
C programming Basics
Data Structures basics
Compilation of code on Linux / Unix using gcc etc
Same principles can be used to code on windows as well using the winsock library.
```
## Network Programming huh ? Telephone Analogy
```
A telephone call over a “telephony network” works as follows:
- Both parties have a telephone installed.
- A phone number is assigned to each telephone.
- Turn on ringer to listen for a caller. 
- Caller lifts telephone and dials a number.
- Telephone rings and the receiver of the call picks it up.
- Both Parties talk and exchange data.
- After conversation is over they hang up the phone.
```
Dissecting the Analogy
A network application works as follows:
An endpoint (telephone) for communication is created on both ends.
An address (phone no) is assigned to both ends to distinguish them from the rest of the network.
One of the endpoints (caller) initiate a connection to the other.
The other end (receiver) point waits for the communication to start.
Once a connection has been made, data is exchanged (talk).
Once data has been exchanged the endpoints are closed (hang up).



In the world of sockets……
Socket() – Endpoint for communication
Bind()  - Assign a unique telephone number.
Listen() – Wait for a caller.
Connect()  - Dial a number.
Accept() – Receive a call.
Send(), Recv() – Talk.
Close() – Hang up. 
The Client – Server model
Server – An entity which is a provider of information.
Client – An entity which is a seeker of information.
Example – Apache is a web server providing web pages (information) and Internet Explorer is a web client which requests those pages from the server.
In the socket programming world almost all communication is based on the Client-Server model.
The Server starts up first and waits for a client to connect to it. After a client successfully connects, it requests some information. The Server serves this information to the client. The client then disconnects and the Server waits for more clients.
A TCP Server – Client Interaction
A UDP Server – Client Interaction
Before we dive deeper …Data Structures
Let us now look at the data structures used 
to hold all the address Information:

Struct sockaddr {	unsigned short sa_family; 	char sa_data[14];}
Struct sockaddr_in {	short sin_family;	unsigned short sin_port; // Port Number	struct in_addr sin_addr; // IP Address	char sin_zero[8];}
Struct in_addr {	unsigned long s_addr; // 4 bytes long }	

Before we dive deeper……Byte Ordering 
Byte ordering or Endianess is the attribute of a system which indicates whether integers are stored / represented left to right or right to left.
Example 1: short int x = 0xAABB (hex)This can be stored in memory as 2 adjacent bytes as either (0xaa , 0xbb) or as (0xbb, 0xaa). Big Endian:Byte Value :     [0xAA] [0xBB]Memory      :     [  0    ] [   1    ] Little Endian:Byte Value :     [0xBB] [0xAA]Memory      :     [  0    ] [   1   ]

Before we dive deeper ….Byte Ordering
Example 2: int x = 0xAABBCCDDThis 4 byte long integer can be represented in the same 2 orderings:Big Endian:Byte Value: [0xAA]  [0xBB] [0xCC] [0xDD]Memory:     [   0    ]  [   1    ] [   2   ]  [   3   ]Little Endian:Byte Value: [0xDD]  [0xCC] [0xBB] [0xAA]Memory:     [   0    ]  [   1    ] [   2   ]  [   3   ]
All Network data is sent in Big Endian format. 
In the networking world we call this representation as Network Byte Order and native representation on the host as Host Byte Order.
We convert all data into Network Byte Order before transmission.
Some utility functions :
Byte Ordering:Host Byte Order to Network Byte Order:		htons() , htonl()Network Byte Order to Host Byte Order:		ntohs() , ntohl()
IP Address format:Ascii dotted to Binary: inet_aton()Binary to Ascii dotted: inet_ntoa()
Many others exist …… explore the man pages :D 
Diving Deeper into the syscalls()
We will now describe the following calls in detail :
Socket()
Bind()
Listen()
Accept()
Connect()
Read() / Send() / Sendto()
Write() / Recv() / Recvfrom()
Close()
Socket() – A Connection Endpoint
This creates an endpoint for a network connection.Int Socket(int doman, int type, int protocol)domain = PF_INET (IPv4 communication)type = SOCK_STREAM (TCP) , SOCK_DGRAM (UDP)protocol = 0 (for our discussion)
Example : socket(PF_INET, SOCK_STREAM, 0);This will create a TCP socket. 
The call returns a socket descriptor on success and -1 on an error. 
Bind() – Attaching to an IP and Port
A server process calls bind to attach itself to a specific port and IP address.Int Bind(int sockfd, struct sockaddr *my_addr, socklen_t addrlen)sockfd = socket descriptor returned by socket()my_addr = pointer to a valid sockaddr_in structure cast as a sockaddr * pointeraddrlen = length of the sockaddr_in structure
Example : struct sockaddr_in my;my.sin_family = PF_INET;my.sin_port = htons(80);my.sin_addr.s_addr = INADDR_ANY;bzero(&my, 8)bind(sock, (struct sockaddr *)&my, sizeof(my));
Listen() – Wait for a connection
The server process calls listen to tell the kernel to initialize a wait queue of connections for this socket.Int Listen(int sock, int backlog)sock = socket returned by socket()backlog = Maximum length of the pending connections queue
Example: Listen(sock, 10);This will allow a maximum of 10 connections to be in pending state.
Accept() – A new connection !
Accept is called by a Server process to accept new connections from new clients trying to connect to the server.Int Accept(int socket, (struct sockaddr *)&client, socklen_t *client_len)socket = the socket in listen stateclient = will hold the new client’s information when accept returnsclient_len = pointer to size of the client structure
Example : struct sockaddr_in client;int len = sizeof(client);Accept(sock, (struct sockaddr *)&client, &len);
Connect() – connect to a service
Connect is called by a client to connect to a server port.Int Connect(int sock, (struct sockaddr *)&server_addr, socklen_t len)sock: a socket returned by socket()server_addr: a sockaddr_in struct pointer filled with all the remote server details and cast as a sockaddr struct pointerlen: size of the server_addr struct
Example:connect(sock, (struct sockaddr *)server_addr, len); 
Send / Recv – Finally Data !!	
Send(), Recv() , Read() , Write() etc calls are used to send and receive data .Int send(int sock, void *mesg, size_t len, int flags)Int recv(int sock, void *mesg, size_t len, int flags)sock = A connected socketmesg = Pointer to a buffer to send/receive data from/in .len = Size of the message bufferflags = 0 (for our purpose)The return value is the number of bytes actually sent/received. 
Example:char send_buffer[1024];char recv_buffer[1024];int sent_bytes;int recvd_bytes;sent_bytes = send(sock, send_buffer, 1024, 0);recvd_bytes = recv(sock, recv_buffer, 1024, 0);
Close() – Bye ..Bye ! 
Close signals the end of communication between a server-client pair. This effectively closes the socket.Int close(int sock)sock = the socket to close
Example :close(sock);
Done ! ! Phew ….
This is all the theory you need to know about socket programming !!!
Finer points will be more clear when we being coding .
So let the games begin ……… :D 
