---
layout: post
date:   2024-03-02 15:33:45 -0600
title: Web Application Multi-Container Definition with Docker Compose
category: learning-experience
show: Web Application Multi-Container with Docker Compose
description: Configured back-end, front-end and database containers to work together for the required-reading web application. 
skills: Docker
--- 

## Objective 

- In an effort to help upgrading the Required-Reading web application for the Operations Department, I took initiative for this small project with the idea to help the web developers stay consistent with their dependencies requirements among different development environents. 

## What did I do? 

The required-reading application is a Node JS application, with the following stacks: 
- Backend: Express JS
- Frontent: React 
- Database: PostgreSQL 


### Defined service for the database 

- The database for this application is hosted on another department's network which we have to connect ourt app to using port-forwarding.
- The problem arised when the Docker host could not see the port where the database was hosted. After a few hours, I discovered about `host.docker.internal: host-gateway`, which exposed the Docker host to the port that we're forwarding to.  
- The `host-gateway` expose combines with the environment variable `PGHOST=host.docker.internal` was the key to solve the port-forwarding problem. 

{% highlight yaml %}
db:
    image: postgres:15-alpine
    restart: unless-stopped
    extra_hosts:
      - "host.docker.internal:host-gateway"
    env_file:
      - .env
    environment:
      - POSTGRES_PASSWORD=${DB_SUPER_USER}
      - POSTGRES_USER=${DB_USER}
      - PGUSER=${DB_USER}
      - PGDATABASE=${DB_DATABASE}
      - PGPORT=${DB_PORT}
      - PGHOST=host.docker.internal

{% endhighlight %}

### Defined service for the backend server 

I then continued with the backend server's service, mounting volumes so that all the node modules could be properly configured. 

{% highlight yaml %}
backend:
 build:
      context: ./required-reading-be/
    command: /required-reading/node_modules/.bin/nodemon src/bin/www.js
    volumes:
      - /my-home-directory/required-reading-be:/required-reading
      - /my-home-directory/required-reading/required-reading-be/node_modules
    env_file:
      - .env
    extra_hosts:
      - "host.docker.internal:host-gateway"
    ports:
      - 5000:5000

    environment:
      - NODE_ENV=development
    depends_on:
      - db
{% endhighlight%}


### Defined service for the frontend server 

Finally, I wrote the service for the frontend 

{% highlight yaml %}
frontend:

    build:
      context: ./required-reading-ui/
    command: npm run dev
    volumes:
      - /my-home-directory/required-reading/required-reading-ui:/required-reading
      - /my-home-directory/lpham/CODE/required-reading/required-reading-ui/node_modules
    depends_on:
      - backend
      - db
    ports:
      - "3000:3000"
{% endhighlight%}

## Summary 
- This small but exciting project provided an opportunity for me to delve into docker compose services, docker, and also to get a deeper understanding of networking.
