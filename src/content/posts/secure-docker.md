---
title: How to secure Docker
published: 2022-08-31
description: Some tips to have a more secure experience while using Docker
tags: [Docker, Security]
category: Docker
draft: false
---
Ok, the other day I was browsing my bookmarks when I came across an interesting post about Docker security. Because of this, I'm going to share some tips for you to secure your Docker. This way, you will feel safer while using Docker to run some dangerous software, preventing access to your host machine.

## Tip #1: Do NOT have outdated containers running in your server.

This applies too to Docker itself. Using software outdated, even when we use containers, is insecure, and can lead to vulnerabilities. Containers talk directly to the kernel, so a kernel related vulnerability which can allow the user to modify the kernel, can lead us to a disaster.

## Tip #2:  Do NOT expose the Docker daemon socket.

"But, if my containers need to comunicate with other containers..." Well, do it at your own risk, buddy. I do not encourage anyone to do this. You can do it, but following good security practices, as seen in the [Docker Documentation](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-socket-option).

## Tip #3: Do NOT run your containers as root user.

Use an unprivileged user, during runtime:

```
docker run -u 4000 alpine
```

Or during build time:

```
FROM alpine
RUN groupadd -r myuser && useradd -r -g myuser myuser
<HERE DO WHAT YOU HAVE TO DO AS A ROOT USER LIKE INSTALLING PACKAGES ETC.>
USER myuser
```

You should also [enable namespace support](https://docs.docker.com/engine/security/userns-remap/#enable-userns-remap-on-the-daemon).

## Tip #4: Do NOT run containers with all capabilities.

For those who don't know what [capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html) are, they are a set of privileges. Docker, by default, runs with only a subset of capabilities. But, you can drop some capabilities (using **\--cap-drop**) to harden your docker containers, or add some capabilities (using **\--cap-add**) if needed. Remember not to run containers with the **\--privileged flag** (this will add ALL Linux kernel capabilities to the container).

The most secure setup is to drop all capabilities **--cap-drop** all and then add only required ones:

```
docker run --cap-drop all --cap-add CHOWN alpine
```

## Tip #5: Do NOT let any container to escalate privileges.

Always run your docker images with **\--security-opt=no-new-privileges** in order to prevent escalate privileges using setuid or setgid binaries:

```
docker run --security-opt=no-new-privileges alpine
```

## Tip #6: Do NOT allow your containers to communicate between them.

By default inter-container communication (**icc**) is enabled - it means that all containers can talk with each other (using docker0 bridged network). This can be disabled by running docker daemon with --icc=false flag.

If icc is disabled (icc=false) it is required to tell which containers can communicate using **\--link=CONTAINER\_NAME** or **ID:ALIAS** option:

```
docker run --link=web alpine
```

## Tip #7: Do NOT forget about using a good security module.

I have seen information about [seccomp](https://docs.docker.com/engine/security/seccomp/) or [AppArmor](https://docs.docker.com/engine/security/apparmor/). This will help you to protect yourself againts vulnerability.

## Tip #8: Do NOT allow your containers to use as much CPU and memory as they can. Set limits.

You can limit memory, CPU, maximum number of restarts (--restart=on-failure:<number of restarts>), maximum number of file descriptors (--ulimit nofile=<number>) and maximum number of processes (--ulimit nproc=<number>).

Refer to the [documentation](https://docs.docker.com/engine/reference/commandline/run/#set-ulimits-in-container---ulimit) for more information about ulimits, or to this [documentation](https://docs.docker.com/config/containers/resource_constraints/) about memory and cpu limits during runtime.

## Tip #9: Do NOT allow your containers to write to your file system if possible.

If you can, use read-only using **\--read-only** flag. For instance:

```
docker run --read-only alpine sh -c 'echo "whatever" > /tmp'
```

If an application inside a container has to save something temporarily, combine **\--read-only** flag with **--tmpfs** like this:

```
docker run --read-only --tmpfs /tmp alpine sh -c 'echo "whatever" > /tmp/file'
```

In Docker-Compose would be something like:

```
version: "3"
services:
  alpine:
    image: alpine
    read_only: true
```

It is possible to do it with volumes:

```
web:
    container_name: web
    image: nginx
    volumes:
      - /home/user/nginx/html:/usr/share/nginx/html:ro
```

This option is also available in runtime:

```
docker run -v volume-name:/path/in/container:ro alpine
```

You can do this using the **--mount** option:

```
docker run --mount source=volume-name,destination=/path/in/container,readonly alpine
```

There are a few useful tools that can help you in various ways. This list is from that article I previously mentioned above:

#### To scan your containers in search of vulnerabilities: 

##### Free

-   [Clair](https://github.com/coreos/clair)
-   [ThreatMapper](https://github.com/deepfence/ThreatMapper)  
    
-   [Trivy](https://github.com/knqyf263/trivy)

##### Commercial

-   [Snyk](https://snyk.io/) **(open source and free option available)**
-   [anchore](https://anchore.com/opensource/) **(open source and free option available)**  
    
-   [JFrog XRay](https://jfrog.com/xray/)  
    
-   [Qualys](https://www.qualys.com/apps/container-security/)  
    

#### To detect secrets in images:

-   [ggshield](https://github.com/GitGuardian/ggshield) **(open source and free option available)**
-   [SecretScanner](https://github.com/deepfence/SecretScanner)  **(open source)**

#### To detect misconfigurations in Docker:

-   [inspec.io](https://www.inspec.io/docs/reference/resources/docker/)
-   [dev-sec.io](https://dev-sec.io/baselines/docker/)  
    
-   [Docker Bench for Security](https://github.com/docker/docker-bench-security)

## Tip #11: Do NOT forget about logs.

Set up an appropriate log level, you'll thank me later. Countless time I've been safe by logs. You should configure the Docker daemon to log events that you would want to review later:

```
docker-compose --log-level info up  
```

## Tip #12: Do NOT forget about linting your Dockerfiles during build time.

It is as easy as following a few good practices:

-   Ensure a **USER** directive is specified.
-   Ensure the base image version is pinned.  
    
-   Ensure the OS packages versions are pinned.  
    
-   Avoid the use of **ADD** in favor of **COPY**.  
    
-   Avoid curl bashing in **RUN** directives.

Aaaaaand that's it, folks. Following this tips, you should be safer while using Docker. Have a nice day, friend.

## Credits

[https://cheatsheetseries.owasp.org/cheatsheets/Docker\_Security\_Cheat\_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)