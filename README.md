docker-wordpress
================

Simple Dockerfile to run [Wordpress](http://wordpress.com/) inside a [Docker](http://docker.io) container.

Usage
-----

Build the image:

```bash
docker build -t ubermuda/docker-wordpress .
```

Or just pull the trusted build:

```bash
docker pull ubermuda/docker-wordpress
```

>The trusted build will be pulled when you use it anyways,
>so you can skip this step if you prefer to wait when it is
>needed.

You might want to have the database and `wp-content` in shared volumes. To do so, you need to create a data container that will only hold data. You just have to run a container with the `-v` option to create volumes:

```bash
docker run \
    --name wordpress-data \
    -v /var/lib/mysql \
    -v /var/www/wordpress/wp-content \
    busybox \
    /bin/true
```

__OR__ you could write a Dockerfile for your data container, for example in a `wordpress-data/`:

```Dockerfile
FROM        busybox
VOLUME      ["/var/lib/mysql", "/var/www/wordpress/wp-content"]
ENTRYPOINT  ["/bin/true"]
```

then build and run it:

```bash
docker build -t ubermuda/docker-wordpress-data wordpress-data/
docker run --name wordpress-data ubermuda/docker-wordpress-data
```

Any way you chose, you can now use your newly created volumes in the wordpress container:

```bash
docker run -d -P \
    --name wordpress \
    --volumes-from wordpress-data \
    ubermuda/docker-wordpress
```

_OR_ in case you did not mount the volumes and just want to quickly experiment:

```bash
docker run -d -P --name wordpress ubermuda/docker-wordpress
```

Note you let the ports exposed by the container to map directly to the ports on your host with the -P.
If you want to have more control you can do:

```bash
docker run -d -p 8084:80 --name wordpress ubermuda/docker-wordpress
```

Now you can go to http://localhost:8084 on your browser! At this point you are done, but feel free to continue
to learn to improve further the configuration.

Configure nginx as a reverse proxy
----------------------------------

Retrieve your container's port:

```bash
docker port wordpress 80
```

Use this configuration in nginx, changing `{{ port }}` and `{{ server_name }}` accordingly:

```nginx.conf
server {
    listen  80;
    server_name {{ server_name }};
    location / {
        proxy_pass http://127.0.0.1:{{ port }}/;
        proxy_redirect     off;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Host $server_name;
    }
}
```

Disclaimer
----------

This image contains default configurations for nginx, php5 and mysql. It is *not* recommended for production. However, if you're willing to contribute better configurations, please open a pull request!