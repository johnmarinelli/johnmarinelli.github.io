---
title: Building A Load Balancer with nginx and Docker
updated: 2015-10-01
---

Load balancing is often one of those things we hear about in courses and don't really get any tangible experience with until or unless we work on a codebase that requires load balancing.
Here is a simple, DIY load balancer written in nginx.  I use Docker because it's much easier to set up an environment.  All you need is Docker and a text editor.

## nginx.conf
First, let's write an empty nginx.conf file:

```
# nginx.conf
http {
}
```

Now, add a server block.  This will be the main 'entry point' for our nginx load balancer.

```
http {
  server {
    listen 80;
    
    location / {
      root /data/www;
    }
  }
}
```

So, the server will listen on port 80.  When home is requested, we set the root to be /data/www.
Next, let's set up a route to our load balancer.

```
http {

  upstream roundrobin {
    server localhost:8080;
    server localhost:8081;
    server localhost:8082;
    server localhost:80;
  }

  server {
    listen 80;
    
    location / {
      root /data/www;
    }

    location /roundrobin/ {
      proxy_pass http://roundrobin/;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $http_host;
      proxy_set_header HTTP_X_FORWARDED_FOR $proxy_add_x_forwarded_for;
    }
  }
}
```
So we've added a couple of new things here: first is the `upstream roundrobin` block.  
TODO: describe upstream

When a request for localhost:80/roundrobin/ is received, nginx will:
- Proxy the request to the upstream roundrobin { } block
- Forward the headers
Then, the `upstream roundrobin` block will distribute requests in a round-robin fashion automatically.  
Nginx has four different load balancing techniques for their open source version, which you can read about HERE.


However, our work isn't quite done.  We still need to create server blocks for ports 8080, 8081, and 8082.  
This is simple enough.  `mkdir conf.d` and create the following files:

```
# conf.d/server2.conf
server {
  listen 8080;
  
  root /data/server2;
}

# conf.d/server3.conf
server {
  listen 8081;
  
  root /data/server3;
}

# conf.d/server4.conf
server {
  listen 8082;
  
  root /data/server4;
}
```


```
http {
  include /etc/nginx/conf.d/*.conf;

  upstream roundrobin {
    server localhost:8080;
    server localhost:8081;
    server localhost:8082;
    server localhost:80;
  }

  server {
    listen 80;

    location / {
      root /data/www;
    }

    location /roundrobin/ {
      proxy_pass http://roundrobin/;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $http_host;
      proxy_set_header HTTP_X_FORWARDED_FOR $proxy_add_x_forwarded_for;
    }
  }
}
```

## Setting up the HTML files

This is pretty easy.  
`mkdir server1 server2 server3 server4`, then
`echo 'one' > server1/index.html`, 
`echo 'two' > server2/index.html`, 
`echo 'three' > server3/index.html`, 
`echo 'four' > server4/index.html`.  The objective is to get a different response each time we `curl localhost:80`.

## Write your Dockerfile

I'm not too focused on the Docker side of things right now, so I'm just going to plop this file down right here and let you read the comments.  It's pretty self explanatory.

```
# Use nginx as the base image
FROM nginx

# Remove the default Nginx configuration file
RUN rm -v /etc/nginx/nginx.conf

# Remove any other default Nginx configuration file
RUN rm -v /etc/nginx/conf.d/*.conf

# Copy server configuration files from current directory
ADD conf.d/* /etc/nginx/conf.d/

# Copy the main configuration file from the current directory
ADD nginx.conf /etc/nginx/

# Add index.html to /data/www
ADD server1/index.html /data/www/

# Create more server directories
RUN mkdir -p /data/server2
RUN mkdir -p /data/server3
RUN mkdir -p /data/server4

# Add other index.html to /data/server
ADD server2/index.html /data/server2/
ADD server3/index.html /data/server3/
ADD server4/index.html /data/server4/

# Append "daemon off;" to the beginning of the configuration
RUN echo "daemon off;" >> /etc/nginx/nginx.conf

# Expose ports
EXPOSE 80

CMD service nginx start
```

## Test out your new load balancer
Build the Docker image:
`docker build -t nginx-static-load-balancer .`
`docker run -it -p 80:80 nginx-static-load-balancer`

Now try `curl`ing your localhost on port 80 sequentially.  The responses should be the contents of the different HTML pages.

___

Hopefully you got some good devops-y knowledge out of this.  The next step would be to take nginx and place it in front of an app server like node.js.  This is commonly used for serving static assets and enhanced security.  The world is yours.
