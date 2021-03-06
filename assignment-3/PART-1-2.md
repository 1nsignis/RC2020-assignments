# Assignment 3: A robust client to download content from one or multiple HTTP servers

* **Backgound and references**
    * **Progamming with TCP Sockets in Java**
    * **HTTP Protocol and How to DOwnload Digital Objects from HTTP Servers**
* **PART I: Networking Programming using TCP Sockets in Java**
* **PART II: A client and server for TCP File transfers**

# Background and initial references

### Programming with TCP Sockets in Java

- You can find a **tutorial on programming with Sockets in Java Language** in **https://docs.oracle.com/javase/tutorial/networking/sockets/**
- Remember that for the work assignmemt you will be particularly focused on the develpment of clients using TCP sockets (supporting HTTP Requests/Responses) because the HTTP  
servers to be used are provided in advance.
- You also have a convenient explanation in the text book of the course: **https://legatheaux.eu/book/cnfbook-pub.pdf, see chapter 5, section 5.3**.
- Complementarily you have these examples for your preliminary tests: **echo-client.java** and **echo-sever.java** (explained below in Part I):  a very simple client/server application implementing an ECHO protocol. 
- In the part I (below) you find the initial guidelines for **"Networking Programming using TCP Sockets in Java"**

### HTTP Protocol and How to Download Digital Objects from HTTP Servers
As you know, HTTP is supported by the TCP transport protocol, and it operates in two basic variants (HTTP/1.0 - implementaing HTTP Request/Response with non-persistet connections, and hTTP/1.1: using persistent connections). Clients that interact with HTTP servers must send correct HTTP requests (with the proper HEADERS), sent as formatted 
requests sent in the TCP connection previous estabished with the server. Clients must be able to receive HTTP responses, processing them according to the HTTP protocol (interpreting the HEADERS and CONTENTS in the RESPONSE).

For the operation of the HTTP protocol you must consider the explanation in the theoretical classes. 
- You can also study the HTTP protocol in the course textbook: **https://legatheaux.eu/book/cnfbook-pub.pdf, see chapter 12**, paying special attention to HTTP requests/responses using RANGE REQUESTS. 
- In the part II (below) you find the initial guildelines for **"Using the HTTP Protocol to Download Digital Objects from a Server"**


# PART I
# Networking Programming using TCP Sockets in Java

## Summary
- Client/Server Model with TCP
- TCP Sockets
- Java Example
- Exercise: File Transfer over TCP

### Client/Server Model
A Client/Server Application has two base autonomous components that can run as processes in the same Host or distributed in two different internetworked Hosts:
- Server: the first to run - usually always running and ready to process requests from the client
- Client: usually started by the user, to request a service from the server

### Client/Server Model with TCP Channels

The following figure represents the typical interaction between a client and server.

<img src="./figures/TCPSockets.png">

### TCP Logical Channels or Connections (or Streams)
- A TCP connection is a logical two-way reliable channel among two processess
- The connection is open by the client, directed towards the server IP address and port,
- The server IP address and port identifies the other extreme of the connection
- It supports two independent, reliable and ordered flow of bytes — one in each direction
- It can be closed at any moment by any of the two communicating processess
- Before any communication can take place, both sides must agree that they want to establish the communicating TCP channel among them

### TCP Sockets
- A TCP connection is established among two TCP Sockets, one in each extreme of the channel
- A client TCP Socket "opens" a connection to the server side TCP Socket - the first opens the connection, the second one accepts it
- A server creates a TCP Socket to accept incoming connections; this socket has a server port and the server IP address
- A client opens or creates the connection by requesting the creation of a local TCP Socket connected to the server TCP Socket

### Example (ECHO Server and Client)
In this simple example the client creates a TCP Socket by connectiong it to the server TCP Socket; the server Socket is identified by the server address and the socket port. Then, the client reads lines from its console and sends them to the server. The server reads the bytes sent by the client and echoes them back to the client. 

### Java Server Code
The code of the server [**EchoServer.java**](./exemplo1/EchoServer.java) is very simple. It just creates a Socket to accept incoming connections in the previously agreed port. Then it accepts client request to establish a connection.
```
import java.io.* ;
import java.net.* ;

public class EchoServer {

   public static final int DEFAULT_PORT = 8000 ;
	
   public static void main(String args[] ) throws Exception {

       int port=DEFAULT_PORT;
       if (args.length == 1) {
	   port=Integer.parseInt(args[0]);
       }
       
       System.out.println("Server waiting in port " + port);
       // creates a server socket to wayt for connections
       try (ServerSocket serverSocket = new ServerSocket( port )) {
	   for(;;) { 
	             Socket clientSocket = serverSocket.accept() ;
		     System.out.println("Got a client connection from "
			+ clientSocket.getInetAddress().getHostName());

		     // handling the client connection
		     new ConnectionHandler().handle( clientSocket );
		   }
       } catch (IOException x) {
	   x.printStackTrace();
       }
   }
}
```

When the connection is established, the handler [**ConnectionHandler.java**](./exemplo1/ConnectionHandler.java) simply continously reads bytes and writes them back to the other side while the connection is not closed.
```
import java.io.*;
import java.net.*;

public class ConnectionHandler {
    private static final int TMP_BUF_SIZE = 1024;

    public void handle(Socket cs) throws IOException {

	InputStream is = cs.getInputStream();
	OutputStream os = cs.getOutputStream();

	for(;;) { 
        // implements the data ECHO, by reading and writing 
        // while the connection is not closed
	    int n ;
	    byte[] buf = new byte[TMP_BUF_SIZE] ;
	    while( (n = is.read(buf)) > 0 )
		os.write( buf, 0, n );
	}
    }
}
```
    


### Java Client

The client [**EchoClient.java**](./exemplo1/EchoClient.java) starts by processing the parameters and opening a connection to the server.
When the connection is open, it starts using it as a read / write stream/pipe.
As you can see (EchoClient) Once the connection is established, the client prepares a Scanner to read bytes from the console (System.in).
Enters a loop where it reads a line, sends it to the server, gets the echo and prints it to the console, until it receives the string "!end". 

```
import java.io.*;
import java.net.*;
import java.util.*;

public class EchoClient {

    public static void main(String[] args ) throws Exception {
		
    if( args.length != 2 ) {
      System.out.println("usage: java EchoClient <serverhost> <serverport>");
      System.exit(0) ;
    }
    
    String server = args[0] ;
    int port = Integer.parseInt(args[1]) ;
		
    Socket socket = new Socket( server, port ) ;
    OutputStream os = socket.getOutputStream();
    InputStream is = socket.getInputStream() ;

    Scanner in = new Scanner( System.in ) ;
    byte[] buf;
    int n;
    String echoRequest;
    
	do {
	    echoRequest = in.nextLine();
	    echoRequest = echoRequest + "\n";

    	    System.out.println("I will send: " +echoRequest);
            os.write( echoRequest.getBytes() );

	    String echoReply = new Scanner(is).nextLine();
            System.out.println("Reply form Server: " +echoReply);

      	   } while( !echoRequest.equals("!quit\n") ) ;

           socket.close() ;
    }
}
```

### Some Recipes

### Class ServerSocket
```
try( ServerSocket ss = new ServerSocket( PORT ) ) {
    ...
        cs = ss.accept();
    ...
}
```

### Class Socket
```
try( Socket ss = new Socket( server, PORT ) ) {
    ...
    InputStream is = ss.getInputStream();
    OutpoutStream os = ss.getOutputStream();
    ...
}
```

### Sending and receiving (multiple) bytes
```
int n;
byte buf = new byte[TMP_BUF_SIZE];
while( (n = is.read( buf )) > 0 )
    os.write( buf, 0, n)
```    
    
### Reading a single byte at each time (slow)
```
InputStream is = cs.getInputStream();
int b = is.read();
```

### WARNING: Anti-Pattern
```
InputStream.available() works with FileInputStream, but does not work with streams that are backed by Sockets.

Socket cs = new Socket( server, port );
InputStream is = cs.getInputStream();
while( is.available() ) {  
};
```

### About Threads 

Threads can be programmed with different options: Lambda Expressions or use of Helper Classes.

### Threads + Lambda Expression

```
new Thread( () -> {
    
    // place here code to execute in new thread...
    
}).start();
```


### Threads + Helper class

```
Helper class implements interface Runnable

Main thread calls:

new Thread( new HelperClass( args )).start();

Child thread executes in run(), receives args in constructor...

  class HelperClass implements Runnable {
    HelperClass( ... ) {
        // Constructor receives any args the helper class needs to run...
    }
    public void run() {
       // place here code to execute in new thread...
    }
}
```
Helper class extends Thread
Cannot be used if helper class already extends another class...

Main thread calls:
new HelperClass( args ).start();


Child thread executes in run(), receives args in constructor...

```
class HelperClass extends Thread {
    HelperClass( ... ) {
        // Constructor receives any args the helper class needs to run...
    }
    public void run() {
       // place here code to execute in new thread...
    }
}
```

### The Multithreaded EchoServer 
This server [**ConcurrentEchoServer.java**](./exemplo1/ConcurrentEchoSerer.java) uses threads to implement concurrency. As you can check the server can handle different clients in parrallel.

```
import java.io.*;
import java.net.*;

public class ConcurrentEchoServer {
    
    public static final int DEFAULT_PORT = 8000 ;
    public static void main( String[] args )throws Exception {

	int port=DEFAULT_PORT;
	if (args.length == 1) {
	    port=Integer.parseInt(args[0]);
	}
        System.out.println("Server waiting in port " + port);

	try (ServerSocket serverSocket = new ServerSocket( port )) {

	    Socket clientSocket;
	    ServiceHandler servthread;

	    while(true) {
		clientSocket = serverSocket.accept();
		servthread = new ServiceHandler(clientSocket);
		servthread.start();
	    }
	}
	catch (IOException x) {
	    x.printStackTrace();
	}
    }
}
```

As you can see in the Concurret EchoServer, the connections from clients are served through the thread **servthread** as defined in [**ServiceHandler.java**](./exemplo1/ServiceHandler.java).
The ServiceHandler class, as an Helper class then uses again the initial [**ConnectionHandler**](./exemplo1/ConnectionHandler.java) used by the non concurrent EchoServer,
but now the client connections are handled in parrallel.

```
import java.io.*;
import java.net.*;

class ServiceHandler extends Thread {
    Socket connection;
	
    public ServiceHandler(Socket c) {
	super("EchoServer service thread");
	connection = c;
    }
    
    public void run()  {
         // handling the client connection                          
	try {
	    new ConnectionHandler().handle(connection);
	}
	catch (IOException x)
	    {
		x.printStackTrace();
	    }
    }
}
```
# PART II
# A Client and Server for file transfers using TCP

## Summary

- The FTTCPServer (File Transfer TCP Server)
- The FTTCPClient (File Transfer TCP Client)

After the last PART I you must be able to understand the code for the **FTTCPServer** and **FTTCPClient**. 

The server [**FTTPTCPServer.java**](./exemplo2/server/FTTCPServer.java) 
accepts TCP connections from clients that will send files. The server saves each file sent in a file named "a.out".
The server computes statistics for end-to-end transfer time, number of transferred bytes and transfer rate.
Notice that the FTTCPServer is not concurrent. It only serves one client for one transfer at a time. If you want, you
can modify the server to work concurrently (following the rationale of the ConsurrentEchoServer in Part I, to serve
multiple clients in parallel.

The client [**FTTPTCPClient.java**](./exemplo2/client/FTTCPClient.java) can send files to the FTTPTCPServer. 

You can use verify the operation trying to transfer files from the client to the server. For example, try to download
the MPEG4 File [**OSIRIS-REx.mp4**](./exemplo2/OSIRIS-REx.mp4) or the JPEG file [**earth.jpg**](./exemplo2.earth.jpg) to test the **FTTCPclient** and **FTTCPServer**.

You can compare the obtained resulsts in your Assignment2 (with your GoBackN and Selective Repeat protocols), 
with the performance you observe here for transfer rates using TCP. 


