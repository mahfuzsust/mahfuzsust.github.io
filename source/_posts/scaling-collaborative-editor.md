---
title: Scale a real-time collaborative editor
date: 2021-07-06 11:43:10
tags:
  - socket
  - node.js
  - ckeditor
categories:
  - System design
---


![alt text](https://miro.medium.com/max/1400/1*7C8YrY6ety7tYjJs3eb9Pg.jpeg "Editor")

In our previous post, we have developed a basic real-time collaborative application.

In this post, we are going to scale the system.

Our previous application is perfectly fine for small project. Let’s say we have added more features into it and we want to deliver this application to end user.

To make this application production ready we need to make sure this is scalable and always available to our customer.

## Vertical scale
It involves upgrading our instance or add extra resource to support increasing workload.

## Horizontal scale
Distributed strategy of adding copies running same task in parallel.

Both of them have separate pros and cons and different use cases based on different scenario. But, in enterprise application horizontal scale is quite popular.

## Socket scale

In our application, we are using socket to handle real-time operations and socket internally uses TCP port to establish a connection. The maximum number of TCP sessions a single source IP can make to a single destination IP and port is 65,535. So, we can only have maximum 65K of connection at a time with a single instance.

There are two ways to solve this issue,
1. Assign IPs to an instance and have ports associated with it. This way we can have maximum connection = number of IP * 65K
2. Running multiple instance.

The problem of assigning IPs to and instance is manual workload with limited capacity. For that reason, we’ll be moving forward with multiple instance. And this way we’ll have the benefit of scaling out more as per need.

With multiple instance, there is another issue of socket synchronization. For an example, two user is connected with two different instance. If one user posts an action, other user will not able to respond to it.
To communicate with these sockets, we need a central place to handle synchronization. Any message broker with pub-sub mechanism will solve. For our use case, we’ll be using redis for this.

{% codeblock %}
...
const { createClient } = require('redis');
const redisAdapter = require('@socket.io/redis-adapter');

const pubClient = createClient({
    host: process.env.REDIS_ENDPOINT || 'localhost',
    port: process.env.REDIS_PORT || 6379
});
if (process.env.REDIS_PASSWORD) {
    pubClient.auth(process.env.REDIS_PASSWORD);
}
const subClient = pubClient.duplicate();
io.adapter(redisAdapter(pubClient, subClient));
...
{% endcodeblock %}

## Load balance

![alt text](https://miro.medium.com/max/1142/1*74Xo4hv3qRN-I38rIZeN6g.png "Load balance")

For load balancing purpose, we’ll be using Nginx as this one of the most popular tools having so much resource. We can also choose other platform like HAProxy or anything preferable.

When we are having multiple instances, we need to make sure our socket sticks to the instance it already connected to. If every time it connects or creates a new socket in each server, then we are going to have performance hit for sure.

{% codeblock %}
events {
    worker_connections  1024;
}
http {
    gzip on;
    gzip_proxied any;
    gzip_types text/plain application/json;
    gzip_min_length 1000;
  
    server {
        listen 80;

        location / {
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $host;

            proxy_pass http://nodes;

            # enable WebSockets
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
    }

    upstream nodes {
        ip_hash;

        server app01:3000;
        server app02:3000;
        server app03:3000;
    }
}
{% endcodeblock %}

To test out our load balancing strategy we are going to run our application using docker compose.

{% codeblock %}
version: '3'
services:
  redis:
    image: redis:alpine

  app01:
    build: .
    environment:
      - REDIS_ENDPOINT=redis
    depends_on:
      - redis
  
  app02:
    build: .
    environment:
      - REDIS_ENDPOINT=redis
    depends_on:
      - redis
  
  app03:
    build: .
    environment:
      - REDIS_ENDPOINT=redis
    depends_on:
      - redis

  web:
    build: nginx/
    ports:
      - 80:80
    depends_on:
      - app01
      - app02
      - app03
{% endcodeblock %}

## Deployment strategy
After figuring out all the tweaks and improvements, we can deploy our service to the cloud infrastructure. There can be separate front end and backend application to support our application.

For a production ready application, checklist of workloads needs to be done before going live.
1. Deploy our front end application behind Content Delivery Network(CDN) which will support scaling across the global regions and caches the content for faster response
2. Deploy our services inside a cluster handling load.
3. Autoscale service container based on parameters
4. Autoscale server instance based on paramerters like CPU usage, Memory 
5. Log and monitor requests and act on regular feedback

## Final product
Let's define some requirement to roll out our application to end users. These are some of the basic features can be added or we'll do a system design for them. 
1. User can authenticate and authorize
2. Realtime collaborative editing
3. Document save and retrieval
4. Document history
5. Comments / feedback
6. Notification

## Front end
Front end can be developed in any framework and deployed behind a CDN. To have better control over the editor, there are many things to consider about. Like
1. Multiple cursor
2. Apply style changes
3. Import / Export support for differnt format

## Backend service
We can think of separate services to support our application.
1. Authentication / Authorization service
2. Application service 
3. Websocket service (Lightweight and Fast)
4. Notification service
5. Comment service

All of these service can be developed and deployed independently behind load balancer. Auto scaling can be done based on different attributes.

## Storage decisions
We have plethora of database solutions to choose from. For out implementation, we can have different combination of selection.

1. RDBMS (PostgreSQL, MySQL etc.) for user management service
2. Document DB(MongoDB, CouchDB, PostgreSQL) for our document storage. 
3. Redis (For publish subscribe message broker)
4. Time series database (InfluxDB) for user operation storage
5. Storage for comments can be added in MongoDB

## Data stream
For our data stream we can use Apache Kafka. This is an open-source distributed event streaming platform.

## Design
![alt text](https://miro.medium.com/max/1400/1*NkGkkbq_MJ0dDvnhvIVRZg.png "Load balance")

## Resource
We tried to design a system based on some requirements. In real case this can change as per need. There are lot of resources can be found over internet to scale socket implementation.

[Socket.io Using multiple nodes](https://socket.io/docs/v3/using-multiple-nodes/index.html)
[Scaling a realtime chat app on AWS using Socket.io, Redis, and AWS Fargate](https://medium.com/containers-on-aws/scaling-a-realtime-chat-app-on-aws-using-socket-io-redis-and-aws-fargate-4ed63fb1b681)
[The Road to 2 Million Websocket Connections in Phoenix — Phoenix Blog](https://phoenixframework.org/blog/the-road-to-2-million-websocket-connections)
[600k concurrent websocket connections on AWS using Node.js](https://blog.jayway.com/2015/04/13/600k-concurrent-websocket-connections-on-aws-using-node-js/)
[10M — goroutines](https://goroutines.com/10m)
