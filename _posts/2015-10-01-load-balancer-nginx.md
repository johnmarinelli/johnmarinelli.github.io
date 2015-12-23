---
title: Building A Load Balancer with nginx and Docker
updated: 2015-10-01
---

DevOps stuff has always eluded me.  This is an attempt to patch over that weakness.

## Install Docker

## Write your Dockerfile
```
FROM nginx

# Remove the default Nginx configuration file
RUN rm -v /etc/nginx/nginx.conf

# Copy a configuration file from the current directory
ADD nginx.conf /etc/nginx/

# Append "daemon off;" to the beginning of the configuration
RUN echo "daemon off;" >> /etc/nginx/nginx.conf

# Expose ports
EXPOSE 80

# Set the default command to execute
# when creating a new container
CMD service nginx start
```

## Write your nginx configuration file
```
events { worker_connections 1024; }

http {

  upstream roundrobin {
    server google.com;
    server msn.com;
    server yahoo.com;
  }


  server {
    listen 80;

    location / {
      proxy_pass http://roundrobin;
    }
  }
}
```

## Test out your new load balancer

