# Add NGINX as a Systemd Service
\*for versions of Ubuntu > 15.0.4

Here you configure NGINX as a systemd service. This will allow you to start, stop, restart, and reload NGINX in a more standard way. More importantly it will allow you to start NGINX on boot. You can enable this by copying a simple script copied from the NGINX website.

#### Terminate nginx please.
```console
# nginx -s stop
```
##### Copy script from NGINX website.
You can find the NGINX init script resource page [here](https://www.nginx.com/resources/wiki/start/topics/examples/initscripts/) and more specifically the example Systemd script [here](https://www.nginx.com/resources/wiki/start/topics/examples/systemd/). Copy the file location.

```console
# touch /lib/systemd/system/nginx.service
```
Go ahead and open that new file with vim, nano, etc., and modify the NGINX example script like the one below. The changes include the location of the PID file as well as the NGINX executable.

  ```console
  [Unit]
  Description=The NGINX HTTP and reverse proxy server
  After=syslog.target network.target remote-fs.target nss-lookup.target

  [Service]
  Type=forking
  PIDFile=/var/run/nginx.pid
  ExecStartPre=/usr/bin/nginx -t
  ExecStart=/usr/bin/nginx
  ExecReload=/usr/sbin/nginx -s reload
  ExecStop=/bin/kill -s QUIT $MAINPID
  PrivateTmp=true

  [Install]
  WantedBy=multi-user.target
  ```

#### Start NGINX using systemd.
```console
# systemctl start NGINX
```

#### Check the status of NGINX using Systemd.
```console
# systemctl status nginx
```

The output should be like the example below.

```console
  ● nginx.service - The NGINX HTTP and reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; disabled; vendor preset: enabled)
     Active: active (running) since Wed 2018-06-06 22:57:12 PDT; 1min 50s ago
    Process: 15403 ExecStart=/usr/bin/nginx (code=exited, status=0/SUCCESS)
    Process: 15393 ExecStartPre=/usr/bin/nginx -t (code=exited, status=0/SUCCESS)
   Main PID: 15404 (nginx)
      Tasks: 2 (limit: 4915)
     Memory: 1.8M
        CPU: 33ms
     CGroup: /system.slice/nginx.service
             ├─15404 nginx: master process /usr/bin/nginx
             └─15405 nginx: worker process

  Jun 06 22:57:12 bae systemd[1]: Starting The NGINX HTTP and reverse proxy server...
  Jun 06 22:57:12 bae nginx[15393]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
  Jun 06 22:57:12 bae nginx[15393]: nginx: configuration file /etc/nginx/nginx.conf test is successful
  Jun 06 22:57:12 bae systemd[1]: Started The NGINX HTTP and reverse proxy server.
```

Notice the third line of the output will give you the status and the last few lines are a few log messages.

#### Terminate NGINX with Systemd.

```console
# systemctl stop nginx
```

#### Configiure NGINX to start on boot.

The command and output are below. This is a one-off command and NGINX will not start on boot.

```console
# systemctl enable nginx
```

Output confirming symlink.
```console
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /lib/systemd/system/nginx.service.
```
