# Rewrites and Redirects


The return statement in the location directive takes a status code and response data.  

`return 200 "Hello, world.";`  

If the status code is in the three-hundred range, then the behavior of the return statement changes in that it accepts a URI as a second parameter. The second parameter being the path the client is being directed.

`return 307 /foo/bar;`

Assuming you have an image asset of some kind, say `thumb.png`, and that asset lives at the root path, NGINX will serve that asset by pointing your browser to the following location.  

`http://<ip_address>/thumb.png`
#### Simple Redirect
Below is a simple example of how you can redirect your client to an asset at the root path.  

```nginx
...

events {}

http {
  include mime.types;
  server {

    listen 80;
    server_name <ip_address>;
    root /sites/nginx-demo;

    # http://<ip_address>/logo
    location /logo {
      return 307 /thumb.png;
    }
  }

...
```
Notice, however, that the browser will take the client to `http://<ip_address>/thumb.png`. This is the main difference between rewrites and redirects.

#### Rewrite With Regular Expression
The following `nginx.conf` file contains a rewrite directive just above `location`. This rewrite directive takes two arguments. The first argument being a regular expression which can be read as, "Starting with `foo` and more than one word character". The second argument of the rewrite directive, `/sup`, will be the new URI to rewrite this to.
```nginx
...

# http://<ip_address>/foo/anything

rewrite ^/foo/\w+ /sup;

location /sup {
  return 200 "Hello from /sup.";
}

...
```
It's important to note here that a rewrite mutates the URI _internally_ which is not visible to the client. The URL is displayed as `http://<ip_address>/foo/anything`, but NGINX is serving what is returned by the prefix match, `/sup`.

#### Capture Groups
With capture groups you can "capture" parts of the original URI. The second argument of the rewrite directive contains the variable, `$1`. Notice how the value of that variable can be returned by both the prefix match and the exact match.

```console
...

# http://<ip_address>/foo/anything
# http://<ip_address>/foo/dude

rewrite ^/foo/(\w+) /sup/$1;

location /sup {
  return 200 "Hello from /sup with variable, $1";
}

location = /sup/dude {
  return 200 "Hello from /sup/dude with variable, $1";
}

...
```

In the above example, `/foo/dude` will be rewritten to `/sup/dude` with the prefix match being skipped in favor of the exact match which takes priority.

#### Optional Flag - `last`
Below is an example of an optional flag that can be used to tell NGINX to stop rewriting.

```console
...

# http://<ip_address>/foo/scooter

rewrite ^/foo/(\w+) /sup/$1;
rewrite ^/sup/scooter /thumb.png;

location /sup {
  return 200 "Hello from /sup with variable, $1";
}

location = /sup/dude {
  return 200 "Hello from /sup/dude with variable, $1";
}

...
```
The above configuration will take the following steps.
1. If `/foo/scooter` is requested, the first rewrite will match it to be evaluated to `sup/scooter`.
2. The second rewrite will then match and then evaluate at `/thumb.png`.

However, if you include an optional flag, `last`, then this will tell NGINX to stop rewriting. The optional flag, `last`, can be seen as part of the first rewrite directive below.
```console
...

# http://<ip_address>/foo/scooter

rewrite ^/foo/(\w+) /sup/$1 last;

...
```
