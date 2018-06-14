# Buffers and Timeouts

For the goal of optimization, this set of directives may be the most difficult to fine tune, i.e., easy to conceptualize, but difficult to perfect. The example configuration below contains a basic set of buffers and timeouts which are perfectly fine to get started with and could even serve as a boilerplate for any given application.

##### Buffering

> The idea here revolves around _reading_ data into memory (RAM) before _writing_ it to the next destination. Buffering is basically a layer of protection between reading and writng data.

##### Timeouts

> A cut-off time for a given event mainly to protect against an endless supply of data which can break the server.

<br>

There is nothing here to actually demonstrate so let the following example configuration serve as a reference for the topics below.

```nginx
user www-data;

worker_processes auto;

events {
  worker_connections 1024;
}

http {
  include mime.types;

  # POST
  client_body_buffer_size 10k;
  client_max_body_size 8m;

  # Headers
  client_header_buffer_size 1k;

  # Max time to receive client headers/body
  client_body_timeout 12;
  client_header_timeout 12;

  # Max time to keep a connection open
  keepalive_timeout 15;

  # Max time for the client accept/receive a response
  send_timeout 10;

  # Skip buffering for static files
  sendfile on;

  # Optimize sendfile packets
  tcp_nopush on;

  server {
    listen 80;
    server_name 198.51.100.0;

    root /sites/nginx-demo;

    index index.php index.html;

    location / {
      try_files $uri $uri/ =404;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php7.1-fpm.sock;
    }

  }
}

```
Before getitng into the buffer directives, please recognize the syntax pattern of buffer directive arguments (buffer size).

* buffer_directive 100 - _bytes_
* buffer_directive 10k - _kilobytes_
* buffer_directive 10m - _megebytes_

### Buffers

`client_body_buffer_size`

The configuration above is set to _allow_ 10 kilobytes for POST submissions. This is a resonable allotment for a basic form submission. Setting this to more than 10 kilobytes essentially wastes space on the machine. Setting this too low will cause NGINX to write part of the buffer to disk. This wastes time and resources.


`client_max_body_size`

The configuration above is set to _reject_ any POST requests larger than 8 megabytes. If this occurs, the server will respond with a 413 - request entity too large.

`client_header_buffer_size`

The configuration above is set to allow 1 kilobyte of memory for request headers.

### Timeouts

`client_body_timeout` & `client_header_timeout`

The two configurations above are set to allow 12 milliseconds for the client body timeout. The is the time set to allow between consecutive read operations to the buffer. An important distinction to make is that this is _NOT_ the time it takes to transfer all the the request data. Both of these directives default to 60 seconds.
 
`keep_alive_timeout`

The configuration above is set to keep the connection open for 15 miliseconds in the event that more data from the client is on the way. If practical, keeping a connection open should be preferred over opening an aditional connection. This directive can have a direct impact on the number of worker_proccesses on the system.

`send_timeout`

The configuration above is set to define the amount of time the server will finally abort the response in the event that the entire request was not received or none of the payload at all.

`sendfile`

The configuration above is set to `on` which tells the server, when sending a file from disk, do not use a buffer, i.e., write the data directly to the response.

`tcp_nopush`

The configuration above is set to `on` which _optimizes_ the size of the files that bypass the buffer.
