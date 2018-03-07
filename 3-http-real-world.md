# HTTP in the Real World

## 1. Deploying to a hosting service
- you can host from home (many do for hobby but not often for job)
- most home computers aren't assigned a *stable (static) IP address* (though you can set that up)
- most home routers don't allow incoming connections (though you can change that)
- most home users don't want to leave their computers on all the time
- yeah, just use Heroku
- Deploy with Heroku!
	1. check in code (since `git push` can deploy your repo code to Heroku)
		- break out the BookmarkServer into its own repo
		- init and commit
	2. sign up for Heroku
	3. install the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli)
		- after this `heroku` will be available in your shell
	4. login through the CLI to authenticate
		- use `heroku login` (will save your info in a hidden `.netrc` file)
	5. create and check in configuration files (`Procfile`, `requirements.txt`, `runtime.txt`)
		- Heroku requires these for deployment in the same dir as your code
		- plain txt files telling servers how to run your app
		- `runtime.txt`: check the currently supported version of Python
		- `requirements.txt`: non-standard Python dependencies (like `requests>=2.12`)
		- `Procfile`: command line for running your app (could be mult srvrs; here `web: python MyApp.py`)
	6. mod server to listen on a configurable port
		- small change to make sure Heroku can tell which port to listen on
		- Heroku uses *environment variable* passed to server from program that starts it (usu the shell)
		- Python can access these through `os.environ`, usu variables are caps like `PORT`
		- make sure to `import os` at top of the file
		- modify the main run to allow for either using PORT if there otherwise 8000
```
if __name__ == '__main__':
	port = int(os.environ.get('PORT', 8000))   # Use PORT if it's there.
	server_address = ('', port)
	httpd = http.server.HTTPServer(server_address, Shortener)
	httpd.serve_forever()
```
	7. create Heroku app with `heroku create my-app`
		- can create with any untaken name
		- will appear at `my-app.heroku.com`
	8. push code to Heroku with `git push heroku master`
		- deploys the app
		- the app should now be web accessible
	9. check logs
		- since it's not running on your machine the logs are elsewhere now
		- navigate to https://dashboard.heroku.com/apps/my-app/logs


## 2. Handling more requests
- no more localhost, now you can send the link to all your friends!
- but there's one more limitation
- Question: What do you think happens if you point the long URI to the app server address itself?
	- Options: sites can't link themselves; need to use the name localhost on itself; server only handles a request at a time
	- Answer: `http.server` can only handle a single request at a time
- **concurrency**: multiple ongoing tasks at same time
	- plug in concurrency to support HTTPServer by adding a Python mixin
	- this *mixin* will add behavior that original class did not have
```
import threading
from socketserver import ThreadingMixIn

class ThreadHTTPServer(ThreadingMixIn, http.server.HTTPServer):
	"This HTTP Server supports thread-based concurrency"

...

if __name__ == '__main__':
	port = int(os.environ.get('PORT', 8000))
	server_address = ('', port)
	httpd = ThreadHTTPServer(server_address, Shortener)
	httpd.serve_forever()
```
- commit this change and push it to Heroku
- now test pointing a long URI to the app itself
- Question: Did it work yet? Or not quite?
	- Answer: Yep!


## 3. What's an Apache or Nginx?
- these servers handle lots of requests quickly
- Apache, Nginx and IIS deliver static content from disk storage very quickly
	- *request routing* or *reverse proxying* for dispatching requests to backend servers
	- *load balancing* for splitting up requests among servers
	- *health check* on backend servers by reverse proxy to send only to up and running servers
	- above also good for allowing some servers to be updated while app runs
- Concurrent users: handling large numbers of users at once
	- complex, involving many requests at once that take time to complete (imagine API, img, ...)
	- *in-flight* requests have taken off from client but response not landed back again on client
	- web service must handle many at once
- Question: Imagine a site receives 250 million page views per day. An average view involves three HTTP queries. Each request takes 0.1 seconds. How many requests are in flight at any instant?
	- Answer: about 2894 * 3 * 0.1 requests
- **Caching**
	- store and serve resource instead of recalculating
	- server sets HTTP headers indicating resource not likely to change much and can be cached
	- governed by *cache control headers* set [by server](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching)
	- caching happens a few places:
		1. *browser cache* like for imgs of recent webpages
		2. *web proxy* if browser configured to pass through one can cache for many users
		3. *reverse proxy* can cache results for a site so not recomputed by slower app server or db
- Capacity
	- Why send results from cache rather than app server? Speed! And handling more requests at once!
	- don't bog down server with traffic; look for quick, efficient software
- Question: Your service is handling 6000 requests a second. 1/3 of requests are for CSS which changes rarely. After you update to have the browser cache the CSS, only 1% of visitors need to fetch it. How many requests will the service get following this update?
	- Answer: about 4000 + 2000 * 0.01


## 4. Cookies
- headers for `'Set-Cookie'` and `'Cookie'` as particularly important for web apps
	- store and transmit cookies
	- cookie refresher: it's a piece of data that web server asks browser to store and send back
	- "immensely important" for many apps
	- e.g. stay logged in
	- e.g. associate multiple queries c a single session
	- e.g. track users for advertising
- **Cookies**: 
	- server asks browser to retain piece of info
	- server requests browser to send it back when browser makes subsequent requests
	- cookie has name and value
	- cookie has rules specifying when it should be sent back
- Why cookies?
	- server can send clients unique cookie values to tell clients apart
	- implement levels on top of HTTP request/response like a *login* or *session*
	- track user activity for ads and marketing
	- sometimes to store user prefs
- How cookies happen
	1. client sends request to server
	2. server sends response with `'Set-Cookie'` header
	3. header contains a cookie *name*, a cookie *value* and some *attributes*
	4. when client makes subsequent requests, browser will send cookie back to server
	5. server can update cookie
	6. server can ask browser to expire cookie
- Cookie fields in your browser
	- check out a cookie in Chrome (look up how to find it if can't)
	- cookie has *name* and *value* (like key value pair); notice no spaces or certain illegal characters
	- value is the "real" data for the cookie such as logged-in user session
	- cookie has *domain* and *path* (its *scope*)
		- by default it's the hostname from the response URI
		- server can set cookie on broader domain
	- the *send for* and *accessible to script* flags are internally *Secure* and *HttpOnly*
		- boolean flags
		- setting Secure only allows cookie to send over HTTPS
		- setting HttpOnly does not allow JavaScript code running on the page to access cookie
	- Last fields deal with cookie lifetime
		- creation time: when response happened that set the cookie
		- expiration time: when server wants browser to stop saving cookie
		- *Expires* and *Max-Age* are two ways server can set expiration time
		- if no expiration field set, cookie expires when browser closes!
- Using Cookies in Python
	- formatting of `'Cookie'` and `'Set-Cookie'` headers is tricky so don't do it manually
	- instead use Python's `http.cookies` module and the `SimpleCookies` class (dict with special behavior)
```
from http.cookies import SimpleCookie, CookieError

out_cookie = SimpleCookie()
out_cookie["dogname"] = "Spadges"
out_cookie["dogname"]["max-age"] = 600
out_cookie["dogname"]["httponly"] = True
```
	- then send cookies from request handler:
```
self.send_header("Set-Cookie", out_cookie["dogname"].OutputString())
```  
	- then read incoming cookies:
```
in_cookie = SimpleCookie(self.headers["Cookie"])
in_data = in_cookie["dogname"].value
```
	- if request does not have valid cookie:
		- `Cookie` header will raise `KeyError` if cookie does not exist
		- `SimpleCookie` constructor will raise `http.cookies.CookieError` if cookie is invalid
	- dealing with cookie security included modded cookies:
		- users can modify cookies even though browsers make this difficult
		- higher lvl kits like Flask and Rails sign cookies so modified ones aren't accepted
		- high-security servers just use cookie to store session ID
		- please `html.escape` special characters if displaying cookie data as HTML
		- there's even more to say [about handling cookies](https://docs.python.org/3/library/http.cookies.html)
- Exercise: server remembers you
	- server asks your name
	- server stores name in cookie on your browser
	- server knows name on revisit
	- starter code: `Lesson-3/2_CookieServer`
	1. in `do_POST` set the cookie fields: value, domain (localhost), max-age
	2. in `do_GET` extract and decode returned cookie value
	3. run cookie server
	4. navigate in browser to localhost server
	5. run the Python test with the server live
	6. check browser cookies for localhost domain to find the cookie
- DNS domains and cookie security
	- remember back to lesson 1, using `host` or `nslookup` to search for IP addresses of domains?
	- domain names aren't just convenient and easier-to-remember 
		- DNS domain links hostname to comp address
		- also indicates that domain owner means for that comp to be treated as part of that domain
	- so think about what someone could do if they convinced your browser their server was part of Facebook
		- get you to request a Facebook url from their server instead of FB
		- your browser could send facebook.com cookies to their server along with that request
		- then they have access to your cookies and potentially your account!


## 5. HTTPS for security
