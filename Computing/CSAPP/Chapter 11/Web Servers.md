Web Servers are a one particular example of types of servers that run over TCP/IP. They are distinguished by, amongst other things, their purpose (to deliver web pages to browsers), and the protocol they use (HTTP).
## Web Basics
Web clients and servers interact using a text-based application-level protocol called **HTTP (hypertext transfer protocol)**. HTTP is built on top of TCP/IP. This means it uses TCP/IP to handle data transmission over the Internet, and instead handles how data is presented to the user.

A Web client (known as a **browser**) opens an Internet connection to a server and requests some **content**. The server responds with the requested content and then closes the connection. The browser reads the content and displays it on the screen.

What distinguishes Web services from other file retrieval services? The main difference is that the Web content can be written in a language called **HTML (hypertext markup language)**. A HTML program contains instructions (tags) that tell the browser how to display its content. For example, the code:

```html
<b> Make me bold! </b>
```

tells the browser to use bold text. 

However, the real power of HTML is that a page can contain hyperlinks to content stored on any Internet host. For instance:

```html
<a href="http://www.cmu.edu/index.html">Carnegie Mellon</a>
```

tells the browser to highlight the text object `Carnegie Mellon` and to the given web address. If the user clicks on the highlighted text object, the browser requests the corresponding HTML file and displays it.

## Web Content
To Web clients and servers, **content** is a sequence of bytes with an associated **MIME (multipurpose internet mail extensions)** type. Figure 11.23 shows some common MIME types:

![](_attachments/Screenshot%202023-09-20%20at%2021.29.30.png)

Web servers provide content to clients in two different ways:

1. Fetch a disk file and return its contents to the client. The disk file is called **static content**. The process of returning the file to the client is known as **serving static content**.
2. Run an executable file and return its output to the client. The output produced by the executable at run time is called **dynamic content**. The process of running the program and returning its output to the client is called **serving dynamic content**.

Every piece of content returned by a Web server is associated with some file that's managed by the server. Each of these files has a unique name called a **URL (Universal Resource Locator)**. 
For instance, the URL:

```
http://www.google.com:80/index.html
```

identifies an HTML file called `/index.html` on Internet host `www.google.com` that is managed by a Web server listening on port `80`. The port number is optional and defaults to the well-known HTTP port `80`. 

When we pass this URL to our browser, the browser follows these rough steps:

> 1. URL Parsing
> 2. DNS Lookup
> 3. Establishing a Connection at the correct port
> 4. Sends a request
> 5. Renders the content
> 6. Closes the connection

So clearly, a URL is only relevant when we have a browser that knows how to use it.

URLs for executable files can include program arguments after the filename. A `?` character separates the filename from its arguments, and each argument is separated by a `&` character. 
For example, the URL:

```
http://bluefish.ics.cs.cmu.edu:8000/cgi-bin/adder?15000&213
```

identifies an executable called `/cgi-bin/adder` that will be called with two argument strings: `15000` and `213`. 

Clients and servers use different parts of the URL during the transaction. For instance, a client uses the prefix:

```
http://www.google.com:80
```

to determine what server to contact, and which port it's listening on. The server uses the suffix:

```
/index.html
```

to find the file on its filesystem and to determine whether the request is for static or dynamic content.

There are several points to understand about how servers interpret the suffix of a URL:

1. There are no standard rule for determining if a URL refers to static or dynamic content. Each server has its own rules for files that it manages.
2. The initial `/` in the suffix does **not** denote the Linux root directory. Rather, it denotes the home directory for whatever kind of content is being requested. For example, a server might be configured such that all static content is stored in `/usr/httpd/html` and all dynamic content is in `/usr/httpd/cgi-bin`.
3. The minimal URL suffix is the `/` character, which all servers expand to some default home page such as `/index.html`. This explains why it's possible to fetch the home page of a site by simply typing a domain name to the browser; the browser appends the missing `/` to the URL and passes it to the server.

## HTTP Transactions
Since HTTP is based on text lines transmitted over Internet connections, we can use the Linux `telnet` program to conduct transactions with any Web server on the Internet. `telnet` has largely been supplanted by `ssh` as a remote login tool, but is handy for debugging servers.

Figure 11.24 shows an example HTTP transaction using `telnet`:

![](_attachments/Screenshot%202023-09-20%20at%2021.48.04.png)

In line 1, we run `telnet` and ask it to open a connection to the AOL Web server. `telnet` prints three lines of output, opens the connection, and waits for us to enter text (line 5). To initiate a transaction, we enter a HTTP request. The server replies with an HTTP response, and closes the connection.

### HTTP Requests
A **HTTP Request** consists of a **request line**, followed by zero or more **request headers**, followed by an empty line that terminates the list of headers. A request line has the form:

```
method URI version
```

HTTP supports a number of different **methods**; the only one that we will discuss is `GET`, which accounts for the majority of HTTP requests. The `GET` method instructs the server to generate and return the content identified by the **URI (uniform resource identifier)**. The URI is the suffix of the corresponding URL, that includes the filename and optional arguments. 

The **version** field in the request line indicates the HTTP version to which the request conforms. 

So to summarise, the request in line 5 asks the server to fetch and return the HTML file `/index.html`.

### HTTP Responses
HTTP responses are similar to HTTP requests. The key idea is that the requested data is returned to the client.

## Serving Dynamic Content

I chose to skip the rest of this section, as it seems a bit beyond what I'm interested in.


