# Requests & Responses

## 1. Introduction
- http is lang web browsers and web servers use to speak to each other
- lesson 1: explore building blocks of http
- lesson 2: write web server and client programs from the ground up and handle html form user input
- lesson 3: learn about web server hosting, cookies and practical aspects
- bridge between scripting in Python and understanding of http
- NOT about installing Apache on a Linux server
	- it's about the low-lvl behaviors underlying high-lvl frameworks
	- it's about the protocol itself
- Requirements: Bash, Python3, Git, `ncat` from Nmap network testing toolkit
	- `brew install nmap`
	- check `ncat` works
		- open two terminals and run `ncat -l 9999` (listening on 9999) in one and `ncat localhost 9999` in other
		- text entered into one should show in the other on press return
		- one of the programs is acting as simple network server, the other as client
	- so far this is TCP not yet HTTP

## 2. Your First Web Server
- **transaction** involves client + server
- browser as client sending a request
- server sending response back
- loading page may take requests to difft servers
	- like if the code on a website on a server then goes and requests a vid from another server
- HTTP created to serve hypertext but now used for so much
- low-lvl actions like `ping` are not using http
- mobile apps may have difft `user interface` but use http under the hood for their web technology

- browser has a lot to handle
- server can be really simple: just handle incoming requests
- Python `http.server`
	- navigate to a directory with some text or images or html
	- run `python3 -m http.server 8000`
	- access `localhost:8000` in a browser
	- now other computers on your local network could access
	- check the server log in your shell to see things that relate somehow to the client request
		- dates and times
		- GET HTTP/1.1
		- GET favicon with a 404
		- GET resources in the directory
		- GET the nonexistent file (see question below) with a 404 error
	- how those relate to what's going on in the browser is what we'll explore in next lesson
- Anatomy:
	- browser sends HTTP request
	- Python program you're running receives request
	- program responds with data
	- browser presents the data to you
	- NOTE: if you have index.html you will see that in the browser instead

- Question: what happens if you access a nonexistent file in that directory through the browser?
	- Answer: Error response 404, but server keeps running
	- story: street sign in Mountain View says LA is 404 miles away, so the joke is "Los Angeles Not Found"

- Defining "Server":
	- a program accepting connections from other programs on the network
- Server waits
- Server runs code like calling a func when a client connects
- Connection is a phone call-like channel over which client and server communicate


## 3. Parts of a UI
- web address is a **Uniform Resource Identifier**
	- URL means "a URI for a resource on the network"
-	for a web dev it's not just text you enter into browser address input field
- Anatomy:
	- **scheme**: how the client is to access the resource
		- "file" URI tells client to access through local filesystem
		- "HTTP"/"HTTPS" URI tells client to access a web server (encryption diff)
		- and [more](http://www.iana.org/assignments/uri-schemes/uri-schemes.xhtml)!
	- **hostname**: name of server to connect to
		- follows after scheme
		- split from scheme by `://`
			- `:` is actually appended to scheme while `//` is prepended to hostname
			- you can see that in a zero-hostname URI, e.g. starting with scheme `mailto:`
		- commonly shortened but in anchor tags must follow scheme
	- **path**: points to specific resource on the server
		- when left out without a slash, browser fills in default slash
		- default (with slash) is the `root`
			- no access to resources on server's entire filesystem just the resource root
		- path-only is a **relative URI reference**
			- no scheme, no hostname
			- browser knows which server
			- browser determines location relative to the current resource
	- other
		- **fragment** after path begins with `#` sign, not to server but browser to jump to el with that id
		- **query** string (usu attrib-val pairs) set off by `?` gets sent to server
		- diagram of [structure of the URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier#Syntax)


## 4. Hostnames and ports
- steps to get to the server the browser will talk to
	- browser extracts hostname from URI
	- browser looks up hostname to find IP address
	- browser finds right port number on IP address
- Hostnames (bc "host" is a computer on a network that hosts services)
	- map to IP address, e.g. http://216.58.194.174/
	- DNS ("a set of servers maintained by [ISPs]") translate the 
		- in the shell use `host` (or `nslookup` which first gives DNS address) to find a hostname in the DNS
	- run `host` on `localhost` to find that it's:
		- `127.0.0.1` (IPv4 localhost)
		- `::1` (abbreviated IPv6 localhost)
		- these point to the computer itself
	- `0.0.0.0` is a code for "every IPv4 address on this computer" incl localhost and regular IP address
- Ports
	- most web addresses don't have that port at the end
	- client figures out port from URI scheme
	- HTTP implies port `80`
	- HTTPS implies port `443`
	- the Python web server ran earlier on non-default `8000`
	- What's the number for?
		- all network traffic is split into **packets** (messages)
		- packet has IP address of sending computer
		- unless it's a low-lvl packet (like `ping`) it has sender and receiver port number
		- "IP addresses distinguish computers; port numbers distinguish *programs* on those computers"
		- server *listens* on that port number to receive connections from clients and run some code
		- the OS then knows to forward client request to server listening on that port
	- OS only lets admin/root listen on ports < 1024, hence we used `8000` not `80`!
	- There's even more to learn [about addresses, ports, http](https://www.udacity.com/course/networking-for-web-developers--ud256)
- Question: which URI with explicit port refers to same resource as `https://en.wikipedia.org/wiki/Fish`?
	- Answer: `https://en.wikipedia.org:443/wiki/Fish`


## 5. HTTP GET requests
- Every request has HTTP verb (method) telling server what client wants to do
- GET requests from browser want a copy of a specific resource
- Take a look back at the logs you got in #2 when you ran the localhost Python server
	- **request line** sent by browser to server
	- line has three parts:
		1. **method** for kind of request made
		2. **path** of resource requested (not whole URI, just path!)
		3. **protocol** of the request (dialect is commonly HTTP/1.1 today)
- Send a request manually
	- make sure that Python localhost server from #2 is up and running
	- run `ncat 127.0.0.1 8000` to connect to and send request to demo server
	- type `GET / HTTP/1.1`
	- type `Host: localhost` and press return twice
	- (keep pressing return and you'll get `broken pipe` error)
- What you get in response:
	- a message including `200 OK`
	- `Date:` (date and time)
	- some HTML


## 6. HTTP GET responses
- look closer at response that fulfills client request (sending back copy of the data)
- compare two responses: `Host: google.com` port 80 vs `Host: www.google.com`
	- first returns `301 Moved Permanently`
- **status line**: whether server understood request, has the resource and how to proceed
	- **status code** numbers encode most of the info
		- 1xx informational (in progress)
		- 2xx success
		- 3xx redirect
		- 4xx client error
		- 5xx server error
	- browsers automatically follow the 301 redirect we got from `google.com`
- **headers**: metadata for the response
	- not shown in browser/client but give info about the response
	- `Content-type` and many diff others used for sending a lot of diff info
	- for example **cookies** set by server sending `Set-Cookie` header
	- for example `Content-type` tells kind of data (and if HTML the encoding it's written in)
	- `Content-Length` lets client reuse connection as soon as first response read
		- uses same connection instead of making client reconnect to server!
- **response body**: follows the blank line after header and includes copy of resource (or error)
- "Be a web server!"
	- listen on port 9999: `ncat -l 9999`
	- connect with a browser to localhost:9999
	- what happens? GET request followed by some info
	- now send `HTTP/1.1 307 Temporary Redirect`
	- then `Location: https://www.google.com/` 
	- return twice, notice client opens the website!
	- also notice that server connection ends and you have to run `ncat` again for another request
- Try the same as above but send back `200 OK` and a piece of `text/plain` of length `50`
	- don't forget the blank line between headers and body
	- the body text appears in the browser!
- That's how you, a human, can act as a hands-on HTTP response server :D

## 7. Congratulations!
- That felt a little sneaky, huh?
- You can do something similar with emails to send fake ones.
- Keep pretending to be a web server and a client by hand. You'll learn lots more.
- BUT code "pretending" to be a web server IS a web server!
- you can write Python code to do what we've been doing manually (upcoming lesson)
- get starter code for exercises from https://github.com/udacity/course-ud303
