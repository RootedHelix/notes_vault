a
## Flask

So currently we are using flask which is a WSGI (Web Server Gateway Interface) framework which fails to handle these things: No concurrency, No sockets, No threading,  No networking

- Flask is meant to be run by the web server
- Flask is a WSGI application generator

Internally Flask handles:
- Routing
- Request parsing
- Response creation
…but **exports a single WSGI callable**.

## Gunicorn

Gunicorn only sees: callable(environ, start_response)

Gunicorn is a WSGI Server. Gunicorn Handles: Opens sockets, Accepts HTTP connections, Parses HTTP, Converts request → `environ`, Calls your WSGI app, Sends the returned bytes back to the client

- gunicorn never touches the buisiness logic

![[gunicorn_vs_flask.jpg]]


Why nginx and not only gunicorn?
- Can technically serve HTTP
- But is **terrible** at static files
- Should never terminate TLS
- Should never face the internet directly

So Nginx remains the **front-facing web server**.

- gunicorn opens a unix socket compared to a typical network socket
- nginx would bind

## NGINX

Nginx is designed to function as both a high-performance web server and a reverse proxy. It is optimized to serve static content extremely efficiently, handle large numbers of concurrent connections, and enforce strict HTTP protocol correctness. Its event-driven architecture, minimal attack surface, and robust request parsing make it an excellent choice for front-facing internet servers, where it can either serve content directly or safely proxy validated requests to backend application servers. 


## Systemd

Systemd is a process manager for the entire stack.  
### Responsibilities:
**Starting services**
- Launches nginx 
- Launches Gunicorn
- sets working directories, users, environment variables 
 **Keeping them alive**
- Restarts Gunicorn if it crashes    
- Restarts Nginx if it fails
- Can enforce restart policies
**Boot-time orchestration**
- Starts services in the correct order
- Ensures networking exists before starting servers
 **Isolation & security**
- Drops privileges
- Restricts filesystem access
- Controls environment exposure
**Logging**
- Captures stdout/stderr
- Centralizes logs via `journalctl`

#### Absolute Path of the Systemd file:
`sudo nvim /etc/systemd/system/pricesynergies.service`

Systemd is a process manager that controls :

Under that umbrella, systemd controls:

- how the process starts
- which user/group it runs as
- where it runs from
- what environment it sees
- how it restarts
- how it shuts down
- where logs go

## Other useful controls (you’ll use these a lot)

|Action|Command|
|---|---|
|Start|`sudo systemctl start pricesynergies`|
|Stop|`sudo systemctl stop pricesynergies`|
|Restart|`sudo systemctl restart pricesynergies`|
|Reload unit file|`sudo systemctl daemon-reload`|
|Enable on boot|`sudo systemctl enable pricesynergies`|
|Disable on boot|`sudo systemctl disable pricesynergies`|
|Logs|`journalctl -u pricesynergies -f`|


## A scheduled Service unit vs Long Running Server Service Unit

> **One service unit = one execution model**

Long-running server? → one service  
Scheduled batch job? → another service  
Background worker? → another service


So Essentially I would need another systemd service unit to be schedule a python script to run everyday to retrieve the new daily data 




## File Locations

- systemd file for gunicorn
`sudo nvim /etc/systemd/system/pricesynergies.service`

- nginx price synergies file
`sudo nvim /etc/nginx/conf.d/pricesynergies.conf`

