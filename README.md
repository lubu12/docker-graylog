# Setup Graylog with nginx as reverse proxy via docker-compose
Setup Graylog with nginx as reverse proxy and persisting data via docker-compose

Reference:

https://docs.graylog.org/en/3.1/pages/installation/docker.html
http://docs.graylog.org/en/2.2/pages/configuration/web_interface.html#nginx

Tested at AWS EC2 - Amazon Linux 2


## Important Note
GRAYLOG_HTTP_EXTERNAL_URI won't be updated to Graylog configuration file via docker environment variable. Instead, a custom configuration file is used to overwrite the bundled Graylog configuration files storing in `/usr/share/graylog/data/config/` inside the Docker container.

```
mkdir -p ./graylog/config
cd ./graylog/config
wget https://raw.githubusercontent.com/Graylog2/graylog-docker/3.1/config/graylog.conf
wget https://raw.githubusercontent.com/Graylog2/graylog-docker/3.1/config/log4j2.xml
```

The newly created directory `./graylog/config/` with the custom configuration files now has to be mounted into the Graylog Docker container.

This can be done by adding an entry to the volumes section of the `docker-compose.yml` file.

### Warning
Graylog is running as USER graylog with the ID `1100` in Docker. That ID need to be able to read the configuration files you place into the container. USER graylog can be created and ownership of the configuration can be set by following commands.

```
sudo groupadd -g 1100 graylog
sudo useradd -s /sbin/nologin -g 1100 -u 1100 graylog
sudo chown -R graylog:graylog ./graylog/config
```

## Update Graylog configuration file
In order to make Graylog accessible from external network interface, `http_external_uri` is needed to be set as the public IP of your instance.

### If you don't want to use nginx reverse proxy...
You can access directly at port 9000. Just put the external domain name or IP at `http_external_url` in the configuration file at `./graylog/config/graylog.conf`
```
http_external_uri = http://YOUR_PUBLIC_IP:9000/
```

### If you want to use nginx reverse proxy...
At `./graylog/config/graylog.conf`
```
http_external_uri = http://127.0.0.1:9000/
```

## Build and run with docker-compose
```
docker-compose up -d
```

## Rebuild the docker image after configuration file is changed
```
docker-compose down
docker-compose up --build -d
```

## Troubleshooting
Graylog web interface can be accessed from the value you set at `http_external_uri`, i.e., `http://YOUR_PUBLIC_IP:9000/` If it is not accessible, try view the log by running the command `docker logs YOUR_GRAYLOG_DOCKER_CONTAINER_NAME -f`. Make sure the Graylog server is up and running. When you run the command `curl -v 127.0.0.1:9000`, you should be able to see something as follow:

```
$ curl -v 127.0.0.1:9000
* Rebuilt URL to: 127.0.0.1:9000/
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 9000 (#0)
> GET / HTTP/1.1
> Host: 127.0.0.1:9000
> User-Agent: curl/7.61.1
> Accept: */*
>
< HTTP/1.1 200 OK
< X-UA-Compatible: IE=edge
< X-Graylog-Node-ID: 3156d2ec-d10a-49fe-a6ad-c89fc2ace329
< Content-Type: text/html
< Date: Sat, 19 Oct 2019 05:19:55 GMT
< Content-Length: 1350
<
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="robots" content="noindex, nofollow">
    <meta charset="UTF-8">
    <title>Graylog Web Interface</title>
...
```
