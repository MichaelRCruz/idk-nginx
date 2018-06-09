# Variables
There are two types of variables.

###### Configuration Variables
```
set $var 'foo':
```

###### NGINX Module Variables
```
$http, $uri, $args
```

#### Directly Imbed Variables in the Response String
Modify your `nginx.conf` file to include a location which returns three NGINX module variables: `$host`, `$uri`, and `$args`.
```nginx
events {}

http {
  include mime.types;
  server {

    listen 80;
    server_name <ip address>;
    root /sites/nginx-demo;

    location /helloworld {
      return 200 "$host\n$uri\n$args";
    }

```
Point your browser to the two URLs below for an understanding of NGINX variables. Note the behavior of `$args`. This is the query parameter variable and by itself it contains the entire query parameter string.

```text
http://<ip address>/helloworld

...

<your ip address>
/helloworld
```
```text
http://<ip address>/helloworld?foo=bar

...

<your ip address>
/inspect
foo=bar
```
#### Create Variation of `$args` Variable
Modify your `nginx.conf` file to reflect the change to the location context.
```nginx
...

location /helloworld {
      return 200 "Name: $arg_name";
}

...
```
Restart NGINX and point your browser to the below example and notice the variable we defined has been returned.
```text
http://<ip address>/helloworld?name=tupac

...

Name: tupac
```

#### Conditionals
Modify your `nginx.conf` to reflect the changes below. The conditional has been placed _outside_ of the location context as [it is discouraged and considered bad practice otherwise](https://www.nginx.com/resources/wiki/start/topics/depth/ifisevil/). The conditional will check for a hypothetical API key in the URI. If the correct API is not provided by the client, an unauthorized status code is returned otherwise `index.html` will be served.

```http://<ip address>?apikey=123```

```nginx
...

if ( $arg_apikey != 123 ) {
  return 401 "Incorrect API key."
}

...
```

#### Set Own Variables in NGINX
Modify your `nginx.conf` as below noting that the module variable, `$date`, will hold an ISO date/time string with the `$mon` variable defined above the conditional.
```nginx
...

set $mon 'No';

if ( $date_local ~ "Monday" ) {
  set $mon 'Yes';
}

location /is_monday {
  return 200 $mon;
}

...
```
