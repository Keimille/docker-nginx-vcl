# Docker-Nginx-Vcl
Repository to experiment with nginx reverse proxy and VCL with Docker Containers

## How it works 
- Run your application on seperate docker container.
- The VCL will query the server directly and cache the data and the NGINX server will query VCL on behalf of the user.

![](/images/vcl_diagram.png)

## Configuration 

### Configure Web Server
You can run what ever web application or api you would like here. The major thing to consider when running in your local Docker environment is to use the `--network="your_network_here"` command so that all containers are on the same network.

### Configure VCL
Edit or create a default.vcl folder. The configuration will look something like the following:
```
vcl 4.0;

backend default {
  .host = "172.17.0.2:80";
}
```
The host should point to your webserver's ip address and port

### Configure NGINX Reverse Proxy
- This repo has a generic ssl and key (not for production). Please consider creating your own if you are not familar with that step. 
- The configuration for reverse proxy should look something like this:
```
server {
    listen 80;
   server_name site1.test;

# Path for SSL config/key/certificate
ssl_certificate /etc/ssl/certs/nginx/site1.crt;
ssl_certificate_key /etc/ssl/certs/nginx/site1.key;
include /etc/nginx/includes/ssl.conf;

    location /home {
       proxy_pass http://172.17.0.3/home;
     }

    location /author-quotes {
       proxy_pass http://172.17.0.3/author-quotes;
     }

}
```

The key thing to note about this configuration is that ==the proxy_pass value should point to your VCL and NOT the webserver.==

When you run this container make sure to run `service nginx start` in the container's CLI

## Validating 

The first validation step would be to navigate to the address of your nginx server and see if it returns your webserver's content. 

- Use curl to return information on request.

![](/images/curl.png)

You should see the server as Nginx and later down see information on varnish

```
< X-Varnish: 8
< Age: 0
< Via: 1.1 varnish (Varnish/6.5)
```
