# Guide to Nginx

## What is Nginx?
Nginx is a very very fast server which can act as a file server, proxy, load-balancer and much else.
For your web server this is extremely handy because it can 1) serve ``.html`` files
and/or 2) delegate traffic as a ***reverse proxy***.

Nginx is pronounced as "engine X". 

## Installing Nginx
In Ubuntu systems you can install Nginx by simply running ``sudo apt-get install nginx``.

**Please note** that Nginx is a web server. That means it will start listening on port
80 by default. The installation might fail if you have anything else listening on port 80.
For instance Tomcat. If you have a Tomcat installation listening on port 80, go to
the configuration file located at ``/etc/tomcat8/server.xml`` and find the _uncommented_
``<Connector ..> </Connector>`` XML tag (probably around line 78). Here you can change
the port to something different than 80 to allow Nginx to install.

## What we will use Nginx for
We won't use Nginx as anything else than a ***reverse proxy***. We call it _reverse_ because it's listening
for connections and **then** forwarding (potentially many times), contrary to a normal proxy which is forwarding
requests once only.

## Configuring Nginx
Nginx contains some default configuration. That configuration is located in
``/etc/nginx/sites-available``. There are two folders containing Nginx configuration:
``sites-available`` and ``sites-enabled``. The configuration you will need is stored in
the file ``sites-available/default``.

Find the default configuration and open it with a text editor.

A server is defined with the ``server { ... }`` syntax and basically means that Nginx is
listening to one socket on your machine. In the default configuration Nginx defines one
server listening on port 80 (default ``HTTP``). Later in the configuration file the
``Location / { ... }`` determines what happens when a client enters the server at
the root path ``/``. By default it simply looks up a ``.html`` file in ``/var/www/html``
on your machine. This is simpel, static ``HTML`` and way too boring for us. We would like 
to proxy our traffic to a much more interesting Java server.

## Setting up a proxy in Nginx
A proxy is simply just a mechanism that forwards traffic from one endpoint to another.
In this case we would like to forward incoming ``HTTP`` traffic from Nginx on port
80 to Tomcat on the same machine at port 8080.

Please note that this server is **not** handling ``HTTPS`` traffic yet. We will get to 
that later.

Per default Nginx is serving simple boring ``.html`` files. That is done in the
``Location / { ... }`` block where it uses the ``try_files`` configuration to
find the files on the server. Instead of sending the files back to the client,
we want to send the request on to Tomcat and get a reply from there.

To change this, we need to **proxy** the connection on to Tomcat. Assuming Tomcat is
listening on port 8080, we simply need to first comment out the line with the
``try_files`` (write ``#`` in front of the line) and write:

    proxy_pass http://127.0.0.1:8080;

This will forward all the traffic to the _local address_ 127.0.0.1 at port 8080.

For this to work you need to restart your nginx server. Do this by writing

    sudo service nginx restart
    
That's it! Now you have a Nginx reverse proxy listening for traffic on port 80 and
forwarding locally on your Digital Ocean to port 8080.

## Securing the HTTP connection
Having a proxy is all well and good. But when you talk to your super top secret server
you don't want anyone to eavesdrop! So this step is about securing the communication
between the client and the server.

When you think about it, the communication between Nginx and Tomcat is actually secure
because it never leaves the server. So the only link we need to secure is the link between
the client sending the HTTP requests and Nginx.

I wrote a guide on how to do it using Let's encrypt in exercise 3 of our network week:

[https://github.com/CphBusCosSem3/week2-network/tree/master/ex/ex3](https://github.com/CphBusCosSem3/week2-network/tree/master/ex/ex3)

Good luck!
