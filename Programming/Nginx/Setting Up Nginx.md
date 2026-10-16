
## Multiple .conf files

- In order for nginx to read from multiple .conf files you have to add the line: 
	 `http {`  
     `//... existing settings ...`  
    `--->include conf.d/*.conf;`  
`}`
	to the `nginx.conf` file

then you would be able to write multiple conf.d files to the nginx directory







## Example 1 (PriceSynergiesProject)

- in the /nginx/conf.d directory you would mkdir /conf.d/pricesynergies.conf and then create your nginx conf in there

server {
    listen 80;
    server_name _; # This catches all traffic to your IP

    location / {
       # Manually adding standard headers since Arch lacks proxy_params file
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_pass http://unix:/home/helix/Projects/Public_Projects/PriceSynergies/pricesynergies.sock;
    }

    # Optional: Serve static files directly for better performance
    location /static {
       alias /home/helix/Projects/Public_Projects/PriceSynergies/frontend;
    }
}


- - - - 
- Next you should make the pricesynergies script be able to connect to the unix socket from gunicorn. 
- Comment out the server block in /nginx/nginx.conf


## Whitelisting on nginx

Here is an example in the .conf

server {  
    listen 80;  
    server_name _;  
  
    # Allow local machine (always good for debugging)  
    allow 127.0.0.1;  
  
    # Allow your specific remote IP (Find it at 'whatsmyip.org')  
    allow YOUR_HOME_IP_ADDRESS;  
  
    # Block everyone else  
    deny all;