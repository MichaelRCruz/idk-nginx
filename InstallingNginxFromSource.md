# Installing NGINX from Source

> This is a quick tutorial on how to install NGINX from source as opposed to  
> installing and maintaining with a package manager, e.g., yum, apt-get, dkpg,  
> etc.. For this simple tutorial I will be using a Linux operating system, namely  
> **Ubuntu Server 17.10**. This method of installation has the advantage of  
including 3rd party modules which can only be included at compile time.  
However, you do miss out on the convenience of security updates with little  
effort.

#### Download NGINX.

Point your browser to [nginx.org](http://nginx.org/en/download.html) and download the current mainline, stable, or legacy  
version using `curl` or `wget`. At the time of this tutorial, we will be using nginx-1.15.0.tar.gz.  

```console
michael@dtla:~$ wget http://nginx.org/download/nginx-1.15.0.tar.gz
```

#### Extract tarball.
```console
michael@dtla:~$ tar -zxvf nginx-1.15.0.tar.gz
```

#### Configure source code for the build (first attempt).

```console
michael@dtla:~/nginx-1.15.0$ ./configure

...

  ./configure: error: the HTTP rewrite module requires the PCRE library.
  You can either disable the module by using --without-http_rewrite_module
  option, or install the PCRE library into the system, or build the PCRE library
  statically from the source with nginx by using --with-pcre=<path> option.
```

#### Install PCRE library for regex, gzip, https, etc.
```console
michael@dtla:~/nginx-1.15.0$ apt-get install libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev
```

#### Run configuration again.
```console
michael@dtla:~/nginx-1.15.0$ ./configure

...

  Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```
#### Reconfigure configuration and set common flags.
```console
michael@dtla:~/nginx-1.15.0$ ./configure

...

  Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```

#### Compile configuration source.
```console
michael@dtla:~/nginx-1.15.0$ make
```

#### Install compiled source.
```console
michael@dtla:~/nginx-1.15.0$ make install
```

#### Check locations.
```console
michael@dtla:~/nginx-1.15.0$ ls -l /etc/nginx/

...

  -rw-r--r-- 1 root root 1077 Jun  6 21:44 fastcgi.conf
  -rw-r--r-- 1 root root 1077 Jun  6 21:44 fastcgi.conf.default
  -rw-r--r-- 1 root root 1007 Jun  6 21:44 fastcgi_params
  -rw-r--r-- 1 root root 1007 Jun  6 21:44 fastcgi_params.default
  -rw-r--r-- 1 root root 2837 Jun  6 21:44 koi-utf
  -rw-r--r-- 1 root root 2223 Jun  6 21:44 koi-win
  -rw-r--r-- 1 root root 5170 Jun  6 21:44 mime.types
  -rw-r--r-- 1 root root 5170 Jun  6 21:44 mime.types.default
  -rw-r--r-- 1 root root 2656 Jun  6 21:44 nginx.conf
  -rw-r--r-- 1 root root 2656 Jun  6 21:44 nginx.conf.default
  -rw-r--r-- 1 root root  636 Jun  6 21:44 scgi_params
  -rw-r--r-- 1 root root  636 Jun  6 21:44 scgi_params.default
  -rw-r--r-- 1 root root  664 Jun  6 21:44 uwsgi_params
  -rw-r--r-- 1 root root  664 Jun  6 21:44 uwsgi_params.default
  -rw-r--r-- 1 root root 3610 Jun  6 21:44 win-utf
```

#### Check for nginx executable and see breakdown of configuration.
```console
michael@dtla:~/nginx-1.15.0$ nginx -V

...

  nginx version: nginx/1.15.0
  built by gcc 7.2.0 (Ubuntu 7.2.0-8ubuntu3.2)
  built with OpenSSL 1.0.2g  1 Mar 2016
  TLS SNI support enabled
  configure arguments: --sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log//nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module
```
