# Worker Processes

When executng the command, `systemctl status nginx`, you are prompted with useful information on the status of the NGINX instance. Below is an example of this output.


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

. . .
```

Two things to note here, from the output above, are the master process and the worker process. The master process is the actual NGINX instance that spawns worker processes with the default number of worker processes being one. These processes can be optimized for the benefit of the application. The optimization can be done through a directive called, `worker_processes`. For an easy demonstration of this, consider the configuration below.

```nginx
user www-data;

worker_processes 2;

events {}

http {

. . .
```

The worker processes directive has been inserted between the `user` directive and the empty `events` directive which, in this case, is taking a single argument, `2`. If you were to reload NGINX with this simple change to the configuration file, you would see two worker process spawned instead of the singular default worker process. Go ahead and test this by restarting NGINX and running `systemctl status nginx`.

```console
. . .

CGroup: /system.slice/nginx.service
             ├─15404 nginx: master process /usr/bin/nginx
             ├─15405 nginx: worker process
             └─15406 nginx: worker process
. . .
```

For a quick output on the number of cores your machine has, run `nproc`. For a more verbose output on your system details, run `lscpu`.

If you prefer to automate the assignment of processes to cores, then simply pass the argument, `auto`, to your worker processes directive like below.

```nginx
user www-data;

worker_processes auto;

events {}

http {

. . .
```

#### Worker Connections

There is a limit to the number of connections each worker process can accept. Check this limit by running `ulimit -n`. The output should be a single interger. Think of this as the number of requests each CPU core can handle at once. So, the configuration for the maximum number of _concurrent_ requests the server can accept is shown below.


```nginx
user www-data;

worker_processes auto;

events {

  worker_connections 1024;
  
}

. . .
```
