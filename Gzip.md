# Gzip Compression

In an effort to further improve static resource responses, you can configure NGINX to compress these responses. If the browser can "advertise" to the server it is capable of interpreting compressed data, then NGINX can use gzip to compress the response and thus reduce the time it takes to send that response to the client.

### Generic Gzip Configuration

```nginx
events {}

http {
    include mime.types;

    gzip on;
    gzip_comp_level 4;
    gzip_types text/javascript;
    gzip_types text/css;

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
}
```
##### `gzip on;`

Enables gzip compression.

##### `gzip_types text/javascript;`

The mime types to set to be compressed.

##### `gzip_comp_level 4;`

The amount of compression to be used. A larger number will yield larger files, but consume less server resources. A smaller number will yield smaller files, but use more server resources. The rate, though, is not linear as the compression becomes less effective after about 3 or 4, i.e., converges to some number I don't know.

##### `add_header Vary Accept-Encoding;`

Don't forget that the client has to "advertise" to the server it can accept compressed data. The location block in the example configures the response based on the client request header, [`Accept-Encoding`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Encoding).

### Test and Inspect the Configuration

Inspect a CSS file and notice the absence of the header, `Content-Encoding`.

```console
# curl -I http://198.51.100.0/style.css

  HTTP/1.1 200 OK
  Server: nginx/1.15.0
  Date: Fri, 15 Jun 2018 06:18:23 GMT
  Content-Type: text/css
  Content-Length: 980
  Last-Modified: Sat, 09 Jun 2018 04:14:00 GMT
  Connection: keep-alive
  ETag: "5b1b5408-3d4"
  Expires: Fri, 15 Jun 2018 07:18:23 GMT
  Cache-Control: max-age=3600
  Cache-Control: public
  Pragma: public
  Vary: Accept-Encoding
  Accept-Ranges: bytes
```

Try again, but this time include a header in the request. Notice the presence of `Content-Encoding: gzip`.

```console
# curl -I -H "Accept-Encoding: gzip" http://198.51.100.0/style.css

  HTTP/1.1 200 OK
  Server: nginx/1.15.0
  Date: Fri, 15 Jun 2018 06:23:24 GMT
  Content-Type: text/css
  Last-Modified: Sat, 09 Jun 2018 04:14:00 GMT
  Connection: keep-alive
  ETag: W/"5b1b5408-3d4"
  Expires: Fri, 15 Jun 2018 07:23:24 GMT
  Cache-Control: max-age=3600
  Cache-Control: public
  Pragma: public
  Vary: Accept-Encoding
  Content-Encoding: gzip
```

### Test the Difference

Download the files.

```console
# curl http://198.51.100.0/style.css > style.css
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  100   980  100   980    0     0  79281      0 --:--:-- --:--:-- --:--:-- 81666

# curl -H "Accept-Encoding: gzip" http://198.51.100.0/style.css > style.min.css
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  100   487    0   487    0     0  42443      0 --:--:-- --:--:-- --:--:-- 44272
```

...and to show off a little.

```console
$ ls -l *.css
  -rw-r--r--  1 tupac  staff  980 Jun 14 23:32 style.css
  -rw-r--r--  1 tupac  staff  487 Jun 14 23:32 style.min.css
```

Both outputs show 487 bytes and 980 bytes for the compressed and uncompressed stylesheets.
