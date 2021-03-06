# alpine-apache

An image for using [Apache][apache], bundled with Alpine Linux and s6.

[![ImageLayers Layers](https://img.shields.io/imagelayers/layers/smebberson/alpine-apache/latest.svg)]()
[![ImageLayers Size](https://img.shields.io/imagelayers/image-size/smebberson/alpine-apache/latest.svg)]()

**_Yet another container for running apache?_**

Yes, but this one is built from [smebberson/alpine-base][alpinebase] that contains [s6][s6] for process management. Small, fast and with [s6][s6].

_**Aren't you only supposed to run one process per container?**_

Hell no! The following are good examples of when multiple processes within one container might be necessary:

- Automatically updating [apache][apache] proxy settings when a down-stream application server (nodejs, php, etc) restarts (and the IP changes).
- Automatically updating [HAProxy][haproxy] configuration to load balance to a group of down-stream application servers.
- Running a logging daemon to centralize log management (i.e. [logentries][logentries], [loggly][loggly], [logstash][logstash]).
- When you need to run a script on application server crash (to tidy something up), as the standard [Docker container restart policies][drsp] won't provide this.

In all of these instances, there is one primary services and secondary support services. When the secondary support services fail, they should be automatically restarted. When the primary service fails, the container itself should restart.

## Versions

- `2.0.0`, `latest` [(Dockerfile)](https://github.com/smebberson/docker-alpine/tree/alpine-apache-v2.0.0/alpine-apache)
- `1.0.0` [(Dockerfile)](https://github.com/smebberson/docker-alpine/tree/alpine-apache-v1.0.0/alpine-apache)

[See VERSIONS.md for image contents.](https://github.com/smebberson/docker-alpine/blob/master/alpine-apache/VERSIONS.md)

## Usage

To use this image include `FROM smebberson/alpine-apache` at the top of your `Dockerfile`, or simply `docker run -p 80:80 -p 443:443 --name apache smebberson/alpine-apache`.

Apache logs (access and error logs) aren't automatically streamed to `stdout` and `stderr`. To review the logs, you can do one of two things:

Run the following in another terminal window:

```
docker exec -i apache tail -f /var/log/apache2/access.log -f /var/log/apache2/error.log
```

or, in your `Dockerfile` symlink the Apaache logs to `stdout` and `stderr`:

```
RUN ln -sf /dev/stdout /var/log/apache2/access.log && \
    ln -sf /dev/stderr /var/log/apache2/error.log
```

## Customisation

This container comes setup as follows:

- [s6][s6] will automatically start apache for you.
- If apache dies, so will the container.

### HTML content

To alter the HTML content that apache serves up (add your website files), add the following to your Dockerfile:

```
ADD /path/to/content /var/www/localhost/
```

htdocs folder with index.html is the default, but that's easily changed (see below).

### Apache configuration

A basic Apache configuration is supplied with this image. But it's easy to overwrite:

- Create your own `httpd.conf`.
- In your `Dockerfile`, make sure your `httpd.conf` file is copied to `/etc/apache/httpd.conf`.

### Restarting apache

[s6][s6] provides [s6-svc][s6-svc] command to control supervised processes. If you're running another process to keep track of something down-stream (for example, automatically updating [apache][apache] proxy settings when a down-stream application server (nodejs, php, etc) restarts) execute the command `s6-svc -h /etc/services.d/apache` to send a `SIGHUP` to apache and have it reload its configuration, launch new worker process(es) using this new configuration, while gracefully shutting down the old worker processes.

### Apache crash

By default, if Apache crashes, the container will stop. This has been configured within `root/etc/services.d/apache/finish`. This is so the host machine can handle any issues, and automatically restart it (the docker way, `docker run --autorestart`).

If you don't want this to happen, simply replace the `root/etc/services.d/apache/finish` with a different file in your image. I like to `ln -s /bin/true /root/etc/services.d/apache/finish` in those instances in which you don't need a finish script and want to have the service restarted by s6.

### Apache configuration

If you need to, you can run a setup script before starting apache. During your `Dockerfile` build process, copy across a file to `/etc/services.d/apache/run` with the following (or customise it as required):

```
#!/usr/bin/env sh

if [ -e ./setup ]; then
./setup
fi

# start apache
exec /usr/sbin/apachectl -DFOREGROUND;
```

[s6]: http://www.skarnet.org/software/s6/
[s6-built-statically]: https://github.com/smebberson/docker-ubuntu-base/blob/master/s6/s6-build
[logentries]: https://logentries.com/
[loggly]: https://www.loggly.com/
[logstash]: http://logstash.net/
[drsp]: https://docs.docker.com/reference/commandline/cli/#restart-policies
[apache]: https://httpd.apache.org/
[haproxy]: http://www.haproxy.org/
[alpinebase]: https://registry.hub.docker.com/u/smebberson/alpine-base/
[s6]: http://www.skarnet.org/software/s6/
[dockerlogs]: https://docs.docker.com/reference/commandline/cli/#logs
