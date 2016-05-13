# traefik-sandbox

This repository is an experiment to see if it is possible to integrate Traefik with minimesos. This repo contains a `minimesosFile` 
which deploys nginx. Traefik is started separately.

# Running

First run Traefik

```
$ sudo ./traefik -c traefik.toml
```

Now start minimesos

```
$ minimesos up
```

The nginx container should be deployed.

# The problem: nginx is not reachable via Traefik

Now try to reach nginx via Traefik

```
$ curl -H "Host:nginx.marathon.localhost" http://127.0.0.1
Bad Gateway
```

Now check the `access.log`

```
127.0.0.1 - - [13/May/2016:11:47:17 +0200] "GET / HTTP/1.1" 502 11 "" "curl/7.46.0" 3 "nginx" "http://172.17.0.5:31999" 303.156µs
```

You can see that it forwards the request to the Mesos agents IP and port, however the nginx container actually listens on the IP of its _own_ network stack on port 80.

Because nothing is listening on port 31999 even though this port was assigned by Mesos, Traefik returns `Bad Gateway`.

# Conclusion

Traefik can only be integrated with minimesos if this networking problem is solved. We can solve it if we use Project Calico. A difficulty for integrating Project Calico into minimesos is 
that Project Calico has to be installed as Docker libnetwork plugin and this is quite intrusive. A way to solve this is to run minimesos as Docker-in-Docker so we have control over the inner Docker daemon
and can install Project Calico. However, running Docker-in-Docker is problematic because of sharing images. Image sharing can be solved if we, on startup, load all images from the host into the registry.

