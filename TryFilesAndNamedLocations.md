# Try Files and Named Locations
`try_files` is a unique NGINX directive in that it can be used inside the `server` context or a `location` context. With this directive, NGINX will search for a resource in any specified locations relative to the root directory. These specified locations exist as arguments for the `try_files` directive with the last argument resulting in a rewrite and reevaluation.

```nginx
events {}

http {
  include mime.types;

  server {
    listen 80;
    server_name <ip_address>;

    root /sites/nginx-demo;

    try_files /scooter.png /sup;

    location /sup {
      return 200 "Sup, yo.";
    }
  }
}
```

In the example above, `try_files` sits within the `server` context which applies to _all_ incoming requests. This directive checks if the asset exists at the path, `/sites/nginx-demo/scooter.png`. If it does not exist, `try_files` will move on to the next argument to check if that asset exists. If `try_files` reaches its last argument, `/sup` in the above example, the argument is treated as an internal rewrite.

#### `try_files` With NGINX Variables
Below, the NGINX variable, `$uri`, has been added as the first argument to the `try_files` directive and another location context has been added as well. Remember, the value of `$uri` is [defined](http://nginx.org/en/docs/http/ngx_http_core_module.html#var_uri) by the current URI in request.

```nginx
...

  try_files $uri /non_existent.png /sup /example_404;

  location /example_404 {
    return 404 "Sorry, my dude. This doesn't exist.";
  }

  location /sup {
    return 200 "Sup, yo.";
  }

...
```
If a request is made with the above configuration for a resource that does not exist, say `http://<ip_address>/bill_and_teds_excellent_adventure`, then NGINX will take the following steps in order. Pay special attention to step 3.

1. Look for the `$uri` as it is, `/sites/nginx-demo/bill_and_teds_excellent_adventure` which doesn't exist thus moving on to the next argument.
2. Look for the asset at the location, `/sites/nginx-demo/non_existent.png` which doesn't exist thus moving on to the next argument.
3. Look for the asset at `/sites/nginx-demo/sup`. Although there is a specified location for it, note that a rewrite is only caused by the last argument in the `try_files` directive. Therefore, moving on to the last argument.
4. Finally, `/sites/nginx-demo/bill_and_teds_excellent_adventure` will be rewritten to `/example_404`.

Check the status code using curl as below.

```console
curl -I http://<ip_address>/bill_and_teds_excellent_adventure

  HTTP/1.1 404 Not Found
  Server: nginx/1.15.0
  Date: Sun, 10 Jun 2018 22:42:43 GMT
  Content-Type: text/plain
  Content-Length: 36
  Connection: keep-alive
```

#### Named Locations

Named locations are simple in that they allow a name assigned to a location context. Below the `@example_404` being the argument that is assigned to the named location, `location @example_404`.
```nginx
...

  try_files $uri /non_existent.png /sup @example_404;

  location @example_404 {
    return 404 "Sorry, my dude. This doesn't exist.";
  }

  location /sup {
    return 200 "Sup, yo.";
  }

...
```
With the above configuration, you can be sure that a reevaluation will not happen on the last argument, but instead a call to that named location.
