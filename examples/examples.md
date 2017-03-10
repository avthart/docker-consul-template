# Examples
For all examples you need to run consul and registrator. Make sure you have boot2docker installed, so that you can run the examples below on your own machine.

Run consul:

```
$ docker run --name consul -d -p 8400:8400 -p 8500:8500 -p 8600:53/udp -h node1 gliderlabs/consul-server -advertise 127.0.0.1 -bootstrap
```

Run registrator:

```
$ docker run \
  --name registrator \
  -d \
  -v /var/run/docker.sock:/tmp/docker.sock \
  -h $HOSTNAME gliderlabs/registrator consul://127.0.0.1:8500
```

nginx
-----
Run nginx container which will route requests to the hello-world container:

```
$ docker run --name nginx -d -p 80:80 -v /etc/nginx/sites-available nginx
```

Run consul template container which generates nginx configuration (modify the location of the nginx.ctml below):

```
$ docker run \
 --name nginx-consul-template \
 -d \
 -e CONSUL_TEMPLATE_LOG=debug \
 -v /var/run/docker.sock:/tmp/docker.sock \
 -v /Users/albertvth/Development/github/docker-consul-template/examples/nginx.ctml:/tmp/nginx.ctmpl \
 --volumes-from nginx \
 avthart/consul-template \
 -consul=127.0.0.1:8500 -wait=5s -template="/tmp/nginx.ctmpl:/etc/nginx/sites-available/default:/bin/docker kill -s HUP nginx"
 ```

 Start a hello-world container:

 ```
 $ docker run --name hello-world-1 -e SERVICE_NAME=hello-world -d -p 80 tutum/hello-world
 ```

Check if you can retrieve the hello-world page:

```
$ curl http://127.0.0.1/
```

When you stop the container you should get a HTTP 502.

Start another hello-world container:

```
$ docker run --name hello-world-2 -e SERVICE_NAME=hello-world -d -p 80 tutum/hello-world
```

This one should be automatically discovered by the nginx-consul-template container.

Check if you can retrieve the hello-world page and you should see a different hostname for every request:

```
$ curl http://127.0.0.1/
```

Clean up containers:

 ```
 $ docker rm -f nginx nginx-consul-template hello-world
 ```
