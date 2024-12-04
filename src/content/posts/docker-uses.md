---
title: Different uses for Docker
published: 2022-08-31
description: More than you think.
tags: [Docker]
category: Docker
draft: false
---
A month ago, I read a post in [Matt Rickard blog](https://matt-rickard.com/non-obvious-docker-uses) about Docker uses barely known among Docker users. So here I am, sharing that knowledge with you, wise reader.

## You can use Docker as a compiler.

[Here](https://docs.docker.com/engine/reference/commandline/build/#custom-build-outputs) is the official documentation about this use case. 

Instead of exporting the build artifacts as a Docker image, you can do it as files on the local filesystem, which can be helpful for producing local binaries, code generation, etc.

For example, we can use docker to export the artifact to a tar file:

```
docker build --output type=local,dest=out .
```

Feel free to visit the documentation for further information.

## You can use Docker build a task-runner alternative to _make._

Matt points in his post that, thanks to Docker Buildkit, you can write alternative frontends to build images, not only Dockerfiles. This functionality is described [here](https://matt-rickard.com/building-a-new-dockerfile-frontend).

## You can use Docker registry to store tarballs.

Because docker registries only store tarballs and metadata, it is easy to set up a place to store configuration.

## You can use git repositories as Docker images

Matt points this possibility in this [post](https://matt-rickard.com/docker-merge/). To sum it up, "Docker merge is a CLI utility that provides a proof-of-concept strategy to merge docker images".

So, you can use that tool to to hijack the "git merge" strategies to natively "merge" docker layers.

## You can use Docker a cross-platform compatibility layer.

The most obvious case for this is the use of Docker Desktop (if you are using this in your company, use Rancher instead).

## You can use Docker to build Linux kernels

You can do it by using [LinuxKit](https://github.com/linuxkit/linuxkit).

Those where the examples mentioned in the post, I hope you find them useful!

## Credits

[Matt Rickard](https://matt-rickard.com/)