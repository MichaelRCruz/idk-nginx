# Headers and Expires

Another strategy for reducing browser load times and limiting the number of requests to the NGINX server, is to cache frequently used assets on the browser. To illustrate the example, consider assets such as CSS files and images. Images for a website do not often change. Why not set an expiration date as to how long that image can remain with the client? Compared to images, other assets like CSS files for example, may be updated more often and thus require a shorter expiration date.

NGINX provides a way to set "expiration dates" through response and request headers for cached assets on the browser.

#### Set a Generic Response Header

```nginx
. . .

    server {
        listen 80;
        server_name 198.51.100.0;

        root /sites/nginx-demo;

        location ~* \.(css|js|jpg|png) {
            add_header Cache-Control public;
            add_header Pragma public;
            add_header Vary Accept-Encoding;
            expires 60m;
        }

    }

. . .
```

##### `Cache-Control public`

This tells the browser that the repsonse _can_ be cached.

##### `Pragma public`

This is the "old school" flavor of `Cache-Control`.

##### `Vary Accept-Encoding`

This tells the browser that the response can be anything based on the `Accept-Encoding` header.

##### `expires 60m`

This sets the expiration date for the response. Please note that the directive's argument can be written in different ways, but result in the same value. For example, sixty minutes (`60m`) is equal to one hour (`1h`) and equal to 3,600 seconds (`3600s`).

Use curl to test your configuration.

```console
# curl -I http://198.51.100.0/scooter.png

  HTTP/1.1 200 OK
  Server: nginx/1.15.0
  Date: Fri, 15 Jun 2018 04:01:39 GMT
  Content-Type: image/png
  Content-Length: 12627
  Last-Modified: Sat, 09 Jun 2018 04:14:00 GMT
  Connection: keep-alive
  ETag: "5b1b5408-3153"
  Expires: Fri, 15 Jun 2018 05:01:39 GMT
  Cache-Control: max-age=3600
  Cache-Control: public
  Pragma: public
  Vary: Accept-Encoding
  Accept-Ranges: bytes
```
