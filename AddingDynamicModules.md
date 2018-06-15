# Adding Dynamic Modules

One last topic on making NGINX more "configurable" is adding dynamic modules. Modules are built directly into the NGINX binary. This means that if additional modules are to be added, then NGINX will have to be rebuilt from source. Dynamic modules are those that can be configured in NGINX _selectively_. This differs from static modules which are always loaded. Please note that any updates to NGINX, be it a security update or updating to a newer version, will require NGINX to be rebuilt from source. This goes for static modules and dynamic modules as well.

If you followed the installation document in this repo, or even by coincidence, the original source and source code can be found in your home directory.

```console
~# ls -l
. . .

  drwxr-xr-x 9 michael michael     4096 Jun  8 21:45 nginx-1.15.0
  -rw-rw-r-- 1 michael michael  1020675 Jun  5 06:55 nginx-1.15.0.tar.gz
  
. . .
```

```console
~/nginx-1.15.0# ls -l

  total 760
  drwxr-xr-x 6 michael michael   4096 Jun  6 20:24 auto
  -rw-r--r-- 1 michael michael 288216 Jun  5 06:47 CHANGES
  -rw-r--r-- 1 michael michael 439474 Jun  5 06:47 CHANGES.ru
  drwxr-xr-x 2 michael michael   4096 Jun  6 20:24 conf
  -rwxr-xr-x 1 michael michael   2502 Jun  5 06:47 configure
  drwxr-xr-x 4 michael michael   4096 Jun  6 20:24 contrib
  drwxr-xr-x 2 michael michael   4096 Jun  6 20:24 html
  -rw-r--r-- 1 michael michael   1397 Jun  5 06:47 LICENSE
  -rw-rw-r-- 1 michael michael    324 Jun  6 21:41 Makefile
  drwxr-xr-x 2 michael michael   4096 Jun  6 20:24 man
  drwxrwxr-x 3 michael michael   4096 Jun  6 21:43 objs
  -rw-r--r-- 1 michael michael     49 Jun  5 06:47 README
  drwxr-xr-x 9 michael michael   4096 Jun  6 20:24 src
```

### Step Zero

In an effort to maintain the current configuration, print to the console the configuration from the current build and copy the configuration flags for reference later.

```console
~/nginx-1.15.0# nginx -V

. . .

configure arguments: --sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log//nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module
```
### Step One

Print a list of modules that exist in this current source code download. Notice the dynamic modules are those that print, `=dynamic`. To filter the same list down to only dynamic modules, run `./configure --help | grep dynamic`.


```console
~/nginx-1.15.0# ./configure --help

. . .

  --with-http_image_filter_module    enable ngx_http_image_filter_module
  --with-http_image_filter_module=dynamic
                                     enable dynamic ngx_http_image_filter_module
  --with-http_geoip_module           enable ngx_http_geoip_module
  --with-http_geoip_module=dynamic   enable dynamic ngx_http_geoip_module
  --with-http_sub_module             enable ngx_http_sub_module

. . .
```

After choosing a dynamic module, .eg., `--with-http_image_filter_module=dynamic`, compile NGINX with the configuration command. Make sure to include the current configuration arguments copied from the previous step. Also, include any dynamic modules from the list above.

Lastly, DO NOT forget to include the modules path, `--modules-path=/etc/nginx/modules`. See the example command below.

*note error in output

```console
~/nginx-1.15.0# ./configure --sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log//nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module --with-http_image_filter_module=dynamic --modules-path=/etc/nginx/modules

. . .

  ./configure: error: the HTTP image filter module requires the GD library.
  You can either do not enable the module or install the libraries.
 ```

Install the http-image-filter module dependency to resolve the error.

```console
# apt-get install libgd-dev
```

Run the configuration command again.

```console
~/nginx-1.15.0# ./configure --sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log//nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module --with-http_image_filter_module=dynamic --modules-path=/etc/nginx/modules

. . .

  Configuration summary
    + using system PCRE library
    + using system OpenSSL library
    + using system zlib library

    nginx path prefix: "/usr/local/nginx"
    nginx binary file: "/usr/bin/nginx"
    nginx modules path: "/etc/nginx/modules"
    nginx configuration prefix: "/etc/nginx"
    nginx configuration file: "/etc/nginx/nginx.conf"
    nginx pid file: "/var/run/nginx.pid"
    nginx error log file: "/var/log/nginx/error.log"
    nginx http access log file: "/var/log//nginx/access.log"
    nginx http client request body temporary files: "client_body_temp"
    nginx http proxy temporary files: "proxy_temp"
    nginx http fastcgi temporary files: "fastcgi_temp"
    nginx http uwsgi temporary files: "uwsgi_temp"
    nginx http scgi temporary files: "scgi_temp"
```

### Step Two

Compile NGINX.

```console
~/nginx-1.15.0# make
```

### Step Three

Install the new source over the existing NGINX install.

```console
~/nginx-1.15.0# make install
```

Verify the configuration.

```console
# nginx -V
```

Reload NGINX.

```console
# systemctl reload nginx
```

Check the status.

```console
# systemctl status nginx

. . .

      Active: active (running) since Thu 2018-06-14 00:42:23 PDT; 16s ago
      
. . .
```

You're golden! 🌟


### Configure the Dynamic Module

Consider the following configuration. Specifically, the location block within it.

```nginx
load_module /etc/nginx/modules/ngx_http_image_filter_module.so;

events {}

http {
    include mime.types;

    server {
        listen 80;
        server_name 198.51.100.0;

        root /sites/nginx-demo;

        location = /scooter.png {
            image_filter rotate 90;
        }
    }
}
```

The dynamic module's `image_filter` will rotate the image 90 degress. The `load_module` directive is how to _selectively_ include the module into the NGINX configuration. Notice the file path specified for the dynamic module via `--modules-path=/etc/nginx/modules`.

Reference the [documentation](http://nginx.org/en/docs/http/ngx_http_image_filter_module.html) on this dynamic module for further configuration options.



