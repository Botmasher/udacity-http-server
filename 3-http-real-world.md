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
