# API Server Example
This project is a simple example about how to use [Node.js](https://nodejs.org/), [Express](http://expressjs.com/),
[PM2](http://pm2.keymetrics.io/) and [Nginx](https://www.nginx.com/) to implement an API server.

```
This project is just an example, a material for learning. So please do not use it in production as it is!
```

<!-- TOC -->

- [API Server Example](#api-server-example)
- [Prerequisites](#prerequisites)
    - [Git](#git)
    - [Node.js 6.x (with npm 3.x)](#nodejs-6x-with-npm-3x)
    - [Nginx](#nginx)
    - [PM2](#pm2)
    - [Clone this project](#clone-this-project)
- [Architectural overview](#architectural-overview)
- [Overview of the used tools](#overview-of-the-used-tools)
    - [Node.js](#nodejs)
        - [Additionl libraries](#additionl-libraries)
    - [PM2](#pm2-1)
    - [Express](#express)
        - [Additional middlewares](#additional-middlewares)
    - [Nginx](#nginx-1)
- [Step by step](#step-by-step)
    - [Server with one process](#server-with-one-process)
        - [Overview](#overview)
        - [Starting the server without PM2](#starting-the-server-without-pm2)
        - [Starting the server with PM2](#starting-the-server-with-pm2)
    - [Server with multiple processes](#server-with-multiple-processes)
        - [Overview](#overview-1)
        - [Starting the server with PM2](#starting-the-server-with-pm2-1)
    - [Server with multiple processes and load balancer](#server-with-multiple-processes-and-load-balancer)
        - [Overview](#overview-2)
        - [Run the servers with multiple processes](#run-the-servers-with-multiple-processes)
        - [Configure Nginx](#configure-nginx)
        - [Start Nginx](#start-nginx)
- [Testing](#testing)
- [Managing the server with PM2](#managing-the-server-with-pm2)
- [Resources](#resources)

<!-- /TOC -->

# Prerequisites
This example requires Linux. If you have a Windows based machine, you can use [Virtual Box](https://www.virtualbox.org/)
to install a Linux. Beside this, you'll need to have the following installed:

## Git
How to install Git: <https://git-scm.com/book/en/v2/Getting-Started-Installing-Git>

## Node.js 6.x (with npm 3.x)
How to install Node.js: <https://nodejs.org/en/download/package-manager/>

## Nginx
How to install Nginx: <https://www.nginx.com/resources/wiki/start/topics/tutorials/install/>

## PM2
After you successfully installed Node.js, you can install PM2 with the following command:
```bash
$ sudo npm install pm2 -g
```

## Clone this project
To be able to play with this example, you need to clone the repository and install its dependencies.
```bash
$ git clone https://github.com/mezeitamas/api-server-example.git
$ cd api-server-example
$ npm install
```

# Architectural overview
The following architecture is the goal, where Nginx is the load balancer, which only proxying the requests to the Node.js servers.
Every box, labeled with *Machine* is a dedicated machine.

![alt text][multiple-processes-and-load-balancer]

However this example is simplified a bit to help to understand the overall picture and play with the example on a single machine.
There will be only one machine and this will run Nginx and the Node.js server (with multiple processes) as well.

![alt text][multiple-processes-and-load-balancer-simplified]

# Overview of the used tools

## Node.js
> As an asynchronous event driven JavaScript runtime, Node is designed to build scalable network applications.

You can read more about Node.js here: <https://nodejs.org/en/about/>

### Additionl libraries
* [moment](https://github.com/moment/moment/)
    > A lightweight JavaScript date library for parsing, validating, manipulating, and formatting dates.
* [winston](https://github.com/winstonjs/winston)
    > A multi-transport async logging library for Node.js.

## PM2
> PM2 is a production process manager for Node.js applications with a built-in load balancer. It allows you to keep
> applications alive forever, to reload them without downtime and to facilitate common system admin tasks.

You can read more about PM2 here: <https://github.com/Unitech/pm2>

## Express
> Express is a minimal and flexible Node.js web application framework that provides a robust set of features for web and
> mobile applications.

You can read more about Express here: <http://expressjs.com/>

### Additional middlewares
The following middlewares are used:
* [helmet](https://github.com/helmetjs/helmet)
    > Helmet helps you secure your Express apps by setting various HTTP headers. It’s not a *silver bullet*, but it can help!
* [csurf](https://github.com/expressjs/csurf)
    > Node.js CSRF protection middleware.
* [body-parser](https://github.com/expressjs/body-parser)
    > Node.js body parsing middleware. Parse incoming request bodies in a middleware before your handlers, available under the req.body property.
* [cookie-parser](https://github.com/expressjs/cookie-parser)
    > Parse Cookie header and populate req.cookies with an object keyed by the cookie names. Optionally you may enable signed cookie support by passing a secret string, which assigns req.secret so it may be used by other middleware.
* [compression](https://github.com/expressjs/compression)
    > Node.js compression middleware.

## Nginx
> NGINX is open source software for web serving, reverse proxying, caching, load balancing, media streaming, and more.
> It started out as a web server designed for maximum performance and stability. In addition to its HTTP server capabilities,
> NGINX can also function as a proxy server for email (IMAP, POP3, and SMTP) and a reverse proxy and load balancer for HTTP, TCP,
> and UDP servers.

You can read more about Nginx here: <https://www.nginx.com/resources/glossary/nginx/>

# Step by step
Let's achieve our goal step by step.

## Server with one process
First, let's check our example with only one process. This is great for development, but not good for production, because there is no load balancing, etc.

### Overview
![alt text][one-process]

### Starting the server without PM2
```bash
$ node ./bin/api-server-example
```
I've added the *start* npm script to *package.json*, so alternatively you can use the *npm start* command to start the server.
```bash
$ npm start
```

### Starting the server with PM2
Start only 1 process, name it as *api-server* and start watching for changes.
```bash
$ pm2 start ./bin/api-server -i 1 --name api-server --watch
```

![alt text][one-process-console]

## Server with multiple processes

### Overview
![alt text][multiple-processes]

### Starting the server with PM2
Start maximum processes depending on available CPUs and name it as *api-server*. If you want to set manually the process number,
change the 0 to a number, for example 4.
```bash
$ pm2 start ./bin/api-server -i 0 --name api-server --watch
```

![alt text][multiple-processes-console]

## Server with multiple processes and load balancer

### Overview
![alt text][multiple-processes-and-load-balancer-simplified]

Nginx acts as a load balancer and just proxying the requests to our servers.

### Run the servers with multiple processes
You can start the servers like we did before, but there is an easier way to do it. All the configurations can be placed into a config file,
and this can be passed to PM2. You can find a prepared config file here: [conf/pm2.config.json](conf/pm2.config.json)

Just simple run these commands to get the servers up and running:
```bash
$ cd conf
$ pm2 start pm2.config.json
```

![alt text][multiple-processes-and-load-balancer-simplified-console]

### Configure Nginx
Please modify your Nginx configuration and add these fragments to it.

The *upstream* configuration is used to define groups of servers. 
In this example only 2 servers were added and pointing to *localhost* for the sake of simplicity. You can read more on the *upstream* Nginx module here:
<http://nginx.org/en/docs/http/ngx_http_upstream_module.html>
```
upstream api-server-upstream {
    server 127.0.0.1:8888;
    server 127.0.0.1:9999;
    keepalive 64;
}
```

The *location* is used to set configuration depending on a request URI.
You can read more on this here: <http://nginx.org/en/docs/http/ngx_http_core_module.html#location>

In our example Nginx will proxy all requests preformed against the */api/* URI to our Node.js servers, listed in the *upstream*.
```
location /api/ {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header X-NginX-Proxy true;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_max_temp_file_size 0;
    proxy_pass http://api-server-upstream;
    proxy_redirect off;
    proxy_read_timeout 240s;
}
```

You can find an example configuration file here: [conf/nginx.conf](conf/nginx.conf), which based on this PM2 documentation: <http://pm2.keymetrics.io/docs/tutorials/pm2-nginx-production-setup>
If you want to use it, you may need to modify it, based on your environment. The default configuration can be found under the */etc/nginx* folder.

### Start Nginx

```bash
$ sudo nginx
```

You can read more about how to use Nginx from the command line: <https://www.nginx.com/resources/wiki/start/topics/tutorials/commandline/>

# Testing
It is really easy to test our server with *curl*. You only need to run the following command:

Without Nginx
```bash
curl --verbose http://localhost:9999/api/accounts
```

With Nginx
```bash
curl --verbose http://localhost/api/accounts
```

You should see something similar on your console:

![alt text][testing-console]

# Managing the server with PM2
Here is a bunch of useful command which you can use to manage your server with PM2.

You can list all the processes and its status.
```bash
$ pm2 list
```

Monitor all processes.
```bash
$ pm2 monit
```

Display all processes logs in streaming.
```bash
$ pm2 logs
```

You can stop the *api-server* processes.
```bash
$ pm2 stop api-server
```

Delete the *api-server* process from PM2 list.
```bash
$ pm2 delete api-server
```

You can find more useful commands in the PM2 cheatsheet: <http://pm2.keymetrics.io/docs/usage/quick-start/#cheatsheet>

# Resources
The following resources are used to create this example.

Express: <http://expressjs.com/>
* Process Managers: <http://expressjs.com/en/advanced/pm.html>
* Security Best Practices: <http://expressjs.com/en/advanced/best-practice-security.html>
* Performance Best Practices: <http://expressjs.com/en/advanced/best-practice-performance.html>

PM2: <http://pm2.keymetrics.io/>
* Documentation: <http://pm2.keymetrics.io/docs/usage/quick-start/>

Nginx: <https://www.nginx.com/>
* Documentation: <http://nginx.org/en/docs/>





[one-process]: ./doc/one-process.png "Server with one process"
[one-process-console]: ./doc/one-process-console.png "Server with one process, PM2 console output"
[multiple-processes]: ./doc/multiple-processes.png "Server with multiple processes"
[multiple-processes-console]: ./doc/multiple-processes-console.png "Server with multiple processes, PM2 console output"
[multiple-processes-and-load-balancer]: ./doc/load-balancer-multiple-machines-and-processes.png "Server with multiple processes and load balancer"
[multiple-processes-and-load-balancer-simplified]: ./doc/load-balancer-multiple-machines-and-processes-simplified.png "Server with multiple processes and load balancer - simplified"
[multiple-processes-and-load-balancer-simplified-console]: ./doc/load-balancer-multiple-machines-and-processes-simplified-console.png "Server with multiple processes and load balancer - simplified, PM2 console output"
[testing-console]: ./doc/testing-console.png "Testing the server with curl, console output"
