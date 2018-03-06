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
- useful narratives: imagine your web app/service is already built; how will users interact with it?
	- this helps imagine the cases you'll need to build for
- **messageboard server** in upcoming exercises
	- user opens main page in browser
	- page displays message input form
	- page displays previous messages
	- submitting form sends request to server
	- server stores submitted message
	- server redisplays main page
- to handle requests install the `requests` module: `pip3 install requests`
- Question: which HTTP methods will the server need to use, and for what?
	- Answer: GET for viewing messages, POST for submitting messages
- Question: not use GET for submitting?
	- Answer: GET will send the message, it will show up in user URL bar, then user might bookmark URL bar, allowing it to be resent every time user visits the main page
- `do_POST` handler method
	- handle a different action when a POST request comes in to server
	- here, the server will see the POST request, send it to `do_POST`, which will store the message in a list, then return all messages seen so far
	- note that unlike GET the POST user data is in the body
	- `self.rfile.read` will read a request (as opposed to `wfile` for writing a response above)
	- `.read` needs to be told how many bytes to read (length of request body)!
	- our code can get length from `Content-length` header
- Headers are strings
	- `self.headers` is instance variable for accessing the HTTP headers
	- case-insensitive header names (keys)
	- values are strings (so `Content-length` may be "140" bytes rather than int 140)
	- since headers might be missing (`KeyError`), instead of accessing ['key'] directly call `.get`
```
data = self.rfile.read(int(self.headers.get('Content-length', 0))).decode()
```
- now use `urllib.parse.parse_qs` to extract POST params
- that should get you started with a `do_POST`!

- Messageboard part 1
	- starter code: `Lesson-2/3_MessageboardPartOne`
	1. find POST request data length
	2. read correct amount of request data
	3. extract message field from request data
	4. Run the .py server
	5. Open the .html file in browser and submit
	6. run the Python test with server running
- Messageboard part 2
	- if you don't want to start off with code from part 1, starter code: `4_MessageboardPartTwo`
	1. Add string variable containing the HTML form from `Messageboard.html` (repeat the data)
	2. Add `do_GET` method encoding and returning the html form
	3. run the server and test in browser (port 8000)
	4. run the Python test with server running
- Question: What happens if you send requests to localhost with diff URI paths (e.g. `sockses/not-boxes`)?
	- Answer: Nothing differs depending on the URI path. It's always that same form.


## 8. Post-Redirect-Get
- many ways to architect apps: single page, client logic, apps mostly on one server, apps across servers
- common **PRG** (Post-Redirect-Get) pattern for server-side apps using forms
	- client POSTs to server to create or update a resource
	- server responsds with `303` (NOT with `200`)
	- redirect causes client to GET the created/updated resource
	- reason: use HTTP methods to accomplish a specific goal
	- example: Wikipedia uses Post-Redirect-Get on page edit
- Messageboard server goals:
	1. go to localhost port in browser
	2. browser GET request to server
	3. server `200` with piece of HTML
	4. user submits form in that HTML
	5. browser sends form through POST to server
	6. server adds/updates data and replies `303` and `Location: ` header
	7. browser requests the initial (redirected) page through GET
	8. server replies with `200` and piece of HTML
- benefit: every page seen by user is result of GET, so can safely bookmark or revisit without resubmit
- Messageboard part 3
	- starter code: `5_MessageboardPartThree`
	1. in `do_POST` send a `303` redirect back to the root page
	2. in `do_GET` put together response data from form template and stored messages
	3. run server and test in browser
	4. run Python test with server running


## 9. Making requests
- nice servers... now write web clients
- `requests` module has [good documentation](http://docs.python-requests.org/en/master/user/quickstart/)
	- if you haven't yet make sure you pip install it
	- Question: if Messageboard server runs on 8000 how would you send a get request using `requests`?
		- Answer: `requests.get("http://localhost:8000")`
- response objects
	- returned objects look like `<Response [200]>`
	- the type of returned objects is `<class 'requests.models.Response'>`
	- Question: if you store a response object in `r`, how can you get the response body?
		- Answer: `r.content` OR `r.text`, though content is for the literal binary response bytes!

- error handling

	- Question: what happens if you access bad URIs using `requests.get`? Try ones with a bad domain, and ones with a good domain but a bad path.
		
		- Answer: a nonexistent site returns Python error (can't save to variable), while nonexistent page on real site gives object with `.status_code == 404` and `.text` is the 404 page
		
		- More detail: some ISPs will try to nonstandardly redirect to advertising page (called DNS hijacking), but not standards-compliant DNS services like Goolge Public DNS


## 10. Using a JSON API
- as a dev you deal with data in many formats
- very often you're writing for a system or using an API and don't get to choose the data format
- JSON really common format for web APIs
- Python code for dealing with JSON
```
res = requests.get("http://swap.co/api/people/1")
res.json()['name']
```
- Question: What happens if you call `.json()` on non-JSON data (like Udacity homepage)?
	- Answer: Python returns a `json.decoder.JSONDecodeError` from Python's `json` library.
- Exercise: extract JSON response data
	1. check out https://uinames.com
	2. use that API to generate profiles e.g. http://uinames.com/api?ext&region=China
	3. use query param `ext` (for showing more fields) and `region` (for country)
	4. starter code: `Lesson-2/6_UsingJSON`
	5. decode JSON data returned by `GET` request
	6. print out the JSON fields in the format specified
	7. run `UINames.py`
	8. run the Python test


## 11. The bookmark server
- finally write code that accepts requests as a server, then makes requests as a client
	- serves up HTML form via a `GET` request
	- accepts form submission via `POST` request
	- checks web addresses using `requests` to make sure they work
	- uses the Post-Redirect-Get design pattern
- Exercise **bookmark server**, a kind of URL shortener
	- starter code: `7_BookmarkServer`
	- `GET` request to `/` displays an HTML form with fields for a long URI and a short name
	- submitting the form sends a `POST` request
	- on `POST` the server looks for two form fields in request body
		- if form fields exist it checks URI with `requests.get` to see if it returns a 200
		- if form field(s) missing server returns 400 saying form field(s) missing
		- if URI exists server stores dict entry mapping short name to long URI
		- if URI exists server then returns HTML page with link to short version
		- if URI does not exist server returns 404
	- on `GET` a short URI, the server looks up to find long URI and serves a redirect
- Steps
	1. write `CheckURI` to take a URI and return `True` if fetch successful else `False`
	2. write `do_GET` to send 303 to a known name
	3. write `do_POST` to send 400 if the form fields are missing from POST
	4. write `do_POST` to send 303 after saving a new URI
	5. write `do_POST` to send 404 if URI fetch does not check out successfully


## 12. You did it!
- it took instructor a lot of tries to get that shortener right
- it looks basic from browser but has lots going on
- if you got yours working, feel proud!
- get a cookie!... a WEB COOKIE in lesson 3 ;D
