# Logging 

There are two log types of logs in NGINX: error logs and access logs. The former is reserved for when things go wrong and the latter for all other activity. With the logging being enabled at defualt, know that these logs can be configured for your benefit. Disabling logs for certain reasons _can_ lead to better performance.

When NGINX was installed then configured, the configuration files were placed in `/var/log/nginx/`.

```
# ls -l /var/log/nginx
total 40
-rw-r--r-- 1 root root 26339 Jun 10 20:36 access.log
-rw-r--r-- 1 root root  7459 Jun 10 20:32 error.log
```

Notice the files sizes in the eample above. You can clear these log files with `echo`. Follow the example below and  
notice the change in file size.

```
# cd /var/log/nginx/
# echo '' > access.log
# echo '' > error.log
# ls -l
total 8
-rw-r--r-- 1 root root 1 Jun 11 22:05 access.log
-rw-r--r-- 1 root root 1 Jun 11 22:05 error.log
```

For a quick demonstration on how this works, point your browser to a URI that does _not_ contain any resources. Don't forget to make sure NGINX is running :). After confirming this task is done by the 404 error message in the browser, head over to the terminal and see what has been logged in `access.log`.

```
# cat access.log
198.51.100.0 - - [11/Jun/2018:22:08:28 -0700] "GET /scooter.png HTTP/1.1" 404 571 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36"
```

Please keep in mind that 404s are _usually_ not errors.

#### Custom Log Files

The following configuration will demonstrate how to use the `access_logs` directive.

```nginx
events {}

http {
    include mime.types;

    server {
        listen 80;
        server_name 198.51.100.0;

        root /sites/nginx-demo;

        location /place {
            access_log /var/log/nginx/access.log
            access_log /var/log/nginx/place.access.log;
            return 200 "Welcome to the place.";
        }
    }
}
```
Restarting NGINX will immediately create the custom log file.

```
# ls -l /var/log/nginx/
total 8
-rw-r--r-- 1 root root 838 Jun 11 22:26 access.log
-rw-r--r-- 1 root root   1 Jun 11 22:05 error.log
-rw-r--r-- 1 root root   0 Jun 11 22:53 place.access.log

```

Point your browser to `http://198.51.100.0/place` then check the contents of the custom access log file, `place.access.log`.

```
# cat /var/log/nginx/place.access.log
198.51.100.0 - - [11/Jun/2018:23:00:39 -0700] "GET /place HTTP/1.1" 200 30 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36"
```

Please know that the log entry above can be found in both the _global_ access log and the _custom_ access log. This is due to the two `access_log` directives within the location block.

#### Disable Logging

Websites with high traffic may require the need to reduce server load. Disabling some logging can help with this. If you would like to disbale access logs for a given location, pass the `off` argument to the access log directive as follows.

```nginx
...

location /place {
    access_log off
    return 200 "Welcome to the place.";
}

...
```
