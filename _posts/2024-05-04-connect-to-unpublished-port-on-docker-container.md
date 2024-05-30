---
layout: post
title: "Oops, I forgot to --publish! How can I connect to the container then?"
tags: docker networking
---

> A fun way to connect over the network to an unpublished port of a process
running inside a Docker container.

---

As we deploy those containers, we typically want to ensure that all relevant behaviours can be reproduced between the artefacts we develop locally and/or test on CI servers and those landing on production servers. Crucially, we should be able to experiment with such artefacts and troubleshoot them.

Sometimes we get too immersed in the development/testing cycle and may realise that we've missed an important step only when it's pretty inconvenient to restart the whole process.

Let's say we have a networked service running inside a Docker container. Here exemplified by [Apache's `httpd`](https://httpd.apache.org) with the following `Dockerfile`:

```dockerfile
FROM httpd@sha256:518e9447a236de47cc2fb4f2dbf06466b6cd5cf50f9951742f9a20af76e8118d

COPY ./public-html/ /usr/local/apache2/htdocs/
```

And `./publish-html/index.html`:

```html
<html>
The answer?
</html>
```

After building the image:

```console
λ docker build -t httpd-fun .
```

We spin up the container:

```console
λ docker run --rm --name httpd-fun httpd-fun
```

However, as we play with the application, we need to make a couple of in-place changes to the container for **experimentation**. Here exemplified by "revealing the answer":

```console
λ docker exec httpd-fun \
    sed -i '/^The answer\?/ s/$/ <strong>42<\/strong>/' /usr/local/apache2/htdocs/index.html
```

After all that, we try to fetch the HTML page from our host machine:

```console
λ curl http://localhost:80
```

And... Oops:

```console
curl: (7) Failed to connect to localhost port 80 after 0 ms: Connection refused
```

It didn't work. There's nothing listening on `localhost:80`!

Of course, that's to be expected. We didn't publish the container's port `80` onto our host and as a result, we couldn't reach the service.

At this point, we could simply re-run the container with the port published, say `-p 8080:80` to expose `80` inside the container to 8080 on our host, or even build a new image with the changes we wanted and start a new container from that. However, we'd lose the changes we've made so far and that might not be acceptable. We'd prefer to keep the changes throughout the experimentation session.

There are different options to proceed (commit the container, shell out to __iptables_ and manually add the necessary rules etc.), but we'll limit ourselves to just two in this blog.

## Connecting to the container's IP

To start off, perhaps the container's IP is routable from the host machine. If that's the case, then we can:

1\. Find the IP of `httpd-fun`:

{% raw %}
```console
λ docker container inspect \
    -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' httpd-fun

=> 172.17.0.2
```
{% endraw %}

2\. Send an HTTP request to the service to its IP as opposed to `localhost`:

```console
λ curl http://172.17.0.2:80
```

3\. Profit:

```html
<html>
The answer? <strong>42</strong>
</html>
```

Although straightforward, that's not always feasible.

## Connecting through a proxy container

Another option is to run a second container with the port published, from which, by [default](https://docs.docker.com/network), there's connectivity with `httpd-fun`, and then proxy connections from the former to the latter:

1\. Find the network name of `httpd-fun` (to ensure there exists connectivity between the containers):

{% raw %}
```console
λ docker container inspect \
    -f '{{range $net,$v := .NetworkSettings.Networks}}{{printf "%s" $net}}{{end}}' \
    httpd-fun

=> bridge
```
{% endraw %}

2\. Run a secondary container with the published port mapped to the host:

```console
λ docker run -it --rm -p 8080:8080 --network bridge \
    --name httpd-fun-proxy alpine@sha256:6457d53fb065d6f250e1504b9bc42d5b6c65941d57532c072d929dd0628977d0 /bin/sh
```

This provides a shell in an alpine container where `8080` on the host goes to `8080` in the container.

3\. For the proxy, we can install [socat](https://linux.die.net/man/1/socat):

```console
# apk update && apk add socat
```

And spin it up:

```console
# socat -v TCP-LISTEN:8080,fork,reuseaddr TCP-CONNECT:172.17.0.2:80
```

This starts `socat`, enables verbose logging, binds a socket at `localhost:8080` for multiple TCP connections, and forwards incoming connections to `172.17.0.2:80` (the container's IP and the port where `httpd` listens).

4\. Send an HTTP request to `localhost:8080`:

```console
λ curl http://localhost:8080
```

5\. Profit:

```html
<html>
The answer? <strong>42</strong>
</html>
```

## Automating the process

We can even take this all a step further and put together a shell script to automate the spinning of the proxy container -- that we can tweak according to our needs:

{% raw %}
```sh
#!/bin/sh

# proxy-container
#
# USAGE
# proxy-container target-container-name target-container-port proxy-container-port

TARGET_CONTAINER_NAME="${1}"
TARGET_CONTAINER_PORT="${2}"
PROXY_CONTAINER_PORT="${3}"

target_container_ip="$(docker container inspect \
	-f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
	${TARGET_CONTAINER_NAME})"

target_container_net="$(docker container inspect \
	-f '{{range $net,$v := .NetworkSettings.Networks}}{{printf "%s" $net}}{{end}}' \
	${TARGET_CONTAINER_NAME})"

docker run -it --rm \
	-p "${PROXY_CONTAINER_PORT}:${PROXY_CONTAINER_PORT}" \
	--network "${target_container_net}" \
	--name "${TARGET_CONTAINER_NAME}-proxy" \
	alpine@sha256:6457d53fb065d6f250e1504b9bc42d5b6c65941d57532c072d929dd0628977d0 \
	/bin/sh -c "apk update && apk add socat && socat -v TCP-LISTEN:${PROXY_CONTAINER_PORT},fork,reuseaddr TCP-CONNECT:${target_container_ip}:${TARGET_CONTAINER_PORT}"
```
{% endraw %}

And use it to achieve the same result we did previously:

```console
λ ./proxy-container httpd-fun 80 8080
```

## Conclusion

We've played with ways to reach out via the network on a container where we'd forgotten to publish a necessary port. This can be particularly helpful during one-shot debugging sessions as part of the broader development process.

To achieve our goal, we've used the versatile `socat`, a powerful tool in our toolbox. As a bonus, we've hacked a script to automate a large portion of the process, which should streamline our experimentation sessions.

That came in handy when I had a service running locally as a Docker container and after doing a couple of experiments with it, I realised that I hadn't published the port. Although there were other means to achieve the same goal, it was entertaining nonetheless.

Finally, it's important to keep track of what we changed and, once the session is over, declare the necessary modifications in the Dockerfile (or similar system). You also need to build a new image with what is necessary to restore immutability and make it go through the standard integration process (checks, reviews, etc) before going to production.
