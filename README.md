# Responsive Crochet Shop Website Deployed w/ Docker, GCP, and Azure

## Description
This Flask-based project is a shop website. The options in the navigation bar are clickable and lead to other pages. I primarily made this project to further practice responsive web design, Flask, and SQL after finishing a [Pokemon Mystery Dungeon Informational Website](https://github.com/dnce17/pkmn_md_site).

This repo's files was cloned from the [crochet_shop_site](https://github.com/dnce17/crochet_shop_site) repo. The crochet shop website was originally deployed at [Render.com](Render.com), but I created this repo to also practice deploying applications with Docker, GCP, and Azure. 

## Original Crochet Shop Website Repo
[crochet_shop_site](https://github.com/dnce17/crochet_shop_site)

## Website Demo
* GCP: https://crochet-shop-website-1057035447975.us-central1.run.app
* Azure: https://crochet-shop-website.greentree-09dfd09f.centralus.azurecontainerapps.io/
* Render: https://crochet-shop-site.onrender.com/
    * Uses free tier, so may take a few seconds to 1 minute to load for the first load

## Features
* Shop page 
    * displays various items in a grid layout
    * has pagination
    * items can be added to cart
* Cart page
    * cart items can be deleted and item quantity can be adjusted

## Notes
The Dockerfile uses the below command since the website is ran using gunicorn
```python
CMD ["gunicorn", "-k", "geventwebsocket.gunicorn.workers.GeventWebSocketWorker", "-w", "1", "-b", "0.0.0.0:8000", "app:app"]
```
### Issue 
Regarding why `-b` and `ip_address:port` (like 0.0.0.0:8000) is needed in the above cmd
* Issue: When running `gunicorn -k geventwebsocket.gunicorn.workers.GeventWebSocketWorker -w 1 app:app` (without -b and ip_address:port), the app is not ran on the port specified in...
```python
if __name__ == "__main__":
    socketio.run(app, debug=True, host="0.0.0.0", port="5000")
```

NOTE: I had already changed the port to 8000 in socketio.run() for app.py. The issue above is referring to the time I had the port at 5000 and ran into the problem.

### Explanation and Solution
#### Key Points
* socketio.run(app, ...): When calling socketio.run(app, ...) inside the if __name__ == "__main__": block, it starts the built-in Flask development server (not gunicorn) and listens on the specified host and port. In the code above, port=5000 is specified, so the Flask development server would runs on port 5000.
* gunicorn: When gunicorn is run, it completely takes over the web server and does not use the Flask development server. It will start the application on the port specified in gunicorn's command (e.g., with the -b option) or the default port (which is 8000 if not specified).

#### Why It Runs on Port 8000
When specifying the port in socketio.run(app, ...) but then running gunicorn, the Flask development server is never actually used. gunicorn starts a separate process and ignores any port settings in socketio.run(). If port with gunicorn is not specified, it will default to port 8000.

In other words:

* socketio.run(app) only applies when running the Flask app directly (i.e., without gunicorn).
* gunicorn overrides the need for socketio.run() and will start the server based on its own configuration.

Using -b and specifying the port like the below allows you to control which port gunicorn will use

* `-b 0.0.0.0:5000`: This binds gunicorn to port 5000 and listens on all IP addresses (0.0.0.0), which is typically what you want for Docker containers.
