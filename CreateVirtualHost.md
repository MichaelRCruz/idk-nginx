# Create a Virtual Host
Create a simple application which consists of an html file, css file, and a static image such as a .png. The idea here is to work with three different examples of assets. We will edit a NGINX configuration file.

#### Create a directory to hold your basic web application.

Within this directory place a simple web application as simple as an index.html paired with a style.css.

```console
  # mkdir /sites/nginx-demo/
```
#### Locate your `nginx.conf` file.
This is the file we will modify to service our web application.
```console
  # ls -l /etc/nginx/

  ...

    -rw-r--r-- 1 root root  193 Jun  8 23:35 nginx.conf

  ...
```
#### Modify the `nginx.conf` file.
This file is currently serving your default holding page. Replace everything within the http, main, and events context to reflect the code snippet below. Note that although the events context is empty, it is still required for the configuration to be valid. The server context is the new virtual host. NGINX assumes port 80 by default. The server_name will be the location in which the server context exists.

`https://<your domain>:80`

```nginx
events {}
http {
  server {
    listen 80;
    server_name 192.168.86.31;
    root /sites/nginx-demo;
  }
}
```

#### Reload NGINX.

The below command will restart NGINX - bringing in the new server context.

```console
  # systemctl restart nginx
```
If the output isn't as expected, then you more than likely have a syntax error in your `nginx.conf` file. You can conveniently check the configuration file manually for errors with the command below. If there are any errors, the output will tell you.

```console
# nginx -t

  nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
  nginx: configuration file /etc/nginx/nginx.conf test is successful
```



#### Set content type for specific file extensions.

The application will load fine in the browser except that the css did not render the document as expected. Note that dev tools will show the `style.css` file has loaded, but did not style the html. This is because NGINX is sending the incorrect mime type for the style sheet. Verify this by requesting the `style.css` headers.
<br>
<br>
Note `Content-Type: text/plain`.

```console
# curl -I http://<domain, subdomain, or ip address>/style.css

  HTTP/1.1 200 OK
  Server: nginx/1.15.0
  Date: Sat, 09 Jun 2018 07:27:17 GMT
  Content-Type: text/plain
  Content-Length: 489
  Last-Modified: Sat, 09 Jun 2018 04:14:00 GMT
  Connection: keep-alive
  ETag: "5b1b5408-3d4"
  Accept-Ranges: bytes
```

Fix this by giving NGINX content types for specific style extensions. Modify the nginx.conf file to include all mime types found in `etc/nginx/mime.types` by adding a new context, `include`. The new configuration file will now look like the one below.

```nginx
events {}
http {
  include mime.types;
#  types {
#    text/html html
#    text/css css
#  }
  server {
    listen 80;
    server_name 192.168.86.31;
    root /sites/nginx-demo;
  }
}
```
Note that the `types` context has been commented out. This is intentional to demonstrate how to specify the mime types _individually_, but it is easier to include all types found at the path `/etc/nginx/mime.types` which exists as a relative path in the configuration file.
<br>
<br>
Restart nginx and check the style sheet's header. It should now read, `Content-Type: text/css`.
