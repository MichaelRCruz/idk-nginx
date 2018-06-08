////////////////////////////////////////////////////////////////////////////////
//                                                                            //
// add nginx as a systemd service                                             //
//                                                                            //
////////////////////////////////////////////////////////////////////////////////

Terminate nginx
sudo nginx -s stop

https://www.nginx.com/resources/wiki/start/topics/examples/initscripts/
systemd
https://www.nginx.com/resources/wiki/start/topics/examples/systemd/

Create systemd Init Script.
michael@bae:~/nginx-1.15.0$ sudo touch /lib/systemd/system/nginx.service

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

Strat nginx.
  michael@bae:~/nginx-1.15.0$ sudo systemctl start nginx
Print a more informative printout of the nginx service status.
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
Terminate nginx.
  michael@bae:~/nginx-1.15.0$ sudo systemctl stop nginx
Configure nginx to startup on boot (once off command) confirmation of startup symlink.
  michael@bae:~/nginx-1.15.0$ sudo systemctl enable nginx
    Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /lib/systemd/system/nginx.service.
