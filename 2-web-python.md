# The Web from Python

## 1. Python's `http.server`
- so far used `http.server` module to run demo server, just demoing module's abilities
- this time build multiple services using the module
- also use `requests` module to act as an HTTP client
- requirements: from here on, OO Python skills as well as repo dl end of last lesson
- Servers, Handlers
	- `http.server` servers have two parts: HTTPServer class and request handler class
	- HTTPServer listens for requests and hands them to request handler
	- **request handler** runs different code for every web service
	1. import `http.server`
	2. subclass `http.server.BaseHTTPRequestHandler` for your **handler class**
	3. define method on handler for each HTTP verb
		- `GET` must be called `do_GET`
		- call built-in handler class methods to read the request and write response
	4. instantiate `http.server.HTTPServer` passing handler class and server info
		- esp pass the port number
	5. call `HTTPServer` instance `run_forever` method
- Access the exercises Lesson-2 subdirectory to find 0_HelloServer
	- run `HelloServer.py` and access localhost port 8000 in a browser
	- open the script and read through
		- note the imports
		- note the subclassed `BaseHTTPRequestHandler`
		- look at the `do_GET` method, which uses `.send_header()`, `.end_headers()` and `wfile.write()`
		- we'll talk about `.encode()` next time
		- the main check sets the server addressm instantiates HTTPServer passing the subclass and serves
	- all that just to say "Hello!" no matter what? So far, yes...


## 2. What about `.encode()`?
- Consider the response body from above:
```self.wfile.write("Hello, HTTP!\n".encode())
```
- that method expects a `bytes` object to write over the network
	- so strings must be encoded
	- `.decode` method turns bytes into string
- older software incl older Python assumes a char takes one byte of mem
- doesn't work well for other langs incl Chinese, esp not multi langs in same string
	- so to send over a binary channel, characters are encoded (now usu `UTF-8`)
	- compare `len('ねこ')` to `len('ねこ'.encode())`


## 3. The echo server
- modify the hello server to use the request it gets
- echo back whatever path you send the server
- scan the [docs](https://docs.python.org/3/library/http.server.html#http.server.BaseHTTPRequestHandler) to find which instance variable holds the path
- open `Lesson-2/1_Echoserver`, rename handler to `EchoHandler` and echo the path
	- a solution: split and encode path in `do_GET` like this: `self.path[1:].encode()`
	- notice that `#fragments` are not echoed back because they stay on the client side!
- Question: What happens if EchoServer listens on port 8000 while HelloServer also does?
	- Answer: Py `OSError: [Errno 48] Address already in use` to new server, which exists (doesn't listen)
	- Answer (WINDOWS 10): 


## 4. Queries and quoting
- look at how query piece of the URI works from server side
- query separated from host on the HTTP server side
- `urllib.parse` for query parameters (see [docs](https://docs.python.org/3/library/urllib.parse.html))
	- check out `urlparse`
	- check out `parse_qs`
- Question: what would `parse_qs("texture=fuzzy&animal=gray+squirrel")` return?
	- Answer: {'texture': ['fuzzy'], 'animal': ['gray squirrel']}
- **Url Quoting**
	- special characters like spaces cannot be in HTTP URL
	- this is more like (and sometimes called) **URL-encoding** or **URL-escaping**
	- spaces sometimes as plus signs
	- others into hexadecimal codes starting with `%`
	- you will need to understand quoting for future project [urllib.parse.quote](https://docs.python.org/3/library/urllib.parse.html#url-quoting)
- keep in mind when you see unexpected spaces or percent signs it could be improper quoting!


## 5. HTML and forms
- query params often from user submitting html form (rather than entered directly into URI)
- open `Lesson-2/2_HTMLForms/LoginPage.html` and start the **echo server** (3 above) listening on 8000
	- open in browser through file system
	- before pressing button to submit login run `EchoServer.py`
	- the server logs a GET with status 200 and the query params
	- after localhost response to GET query params, query is logged in the client
- now open `SearchPage.html` in the same lesson 2 dir
	- fill out the form and submit
	- notice you're now searching on the remote server using these query parms!
- Question: which piece of data tells the browser which server to submit the form to?
	- Answer: the URI in the form `action` attribute!


## 6. GET and POST
- we've been using `GET`, what about `POST`?
- two different HTTP verbs
- altering or creating resources not done with `GET`
- **idempotence**
	- performing the action multiple times produces the same result (like running a search)
	- `POST` requests are not idempotent
	- hence browser mssg when reload on submit form to make sure the form is intentionally resubmitted
- consider some simple idempotent statements:
	- is `x += 0` idempotent?
	- what about `x += 5`?
	- `x = 5`?
	- finally, `h = words['hi']` (a dict lookup)?
- Be a server and receive POST request
	- find the form in `Lesson-2/2_HTMLForms/PostForm.html`
	- open in browser
	- use `ncat` to listen on 9999
	- now add data to and submit form
	- Question: What's different about the HTTP request from the ones seen in this course so far?
		- Answer: request line `POST / HTTP/1.1`, form data not in path, form data elsewhere in response
- note that the data are sent in the request BODY
- also note that the HTTP headers are case sensitive (can do `content-length`, `conTeNt-LeNgTH`, ...)


## 7. A server for POST
