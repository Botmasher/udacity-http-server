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
