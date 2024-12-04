---
title: Docker-Compose Best Practices
published: 2022-08-31
description: I'm not joking, do it, you will thank me later.
tags: [Docker]
category: Docker
draft: false
---
The faster way of doing this is using the CLI. I explained how to do it in my [Self-Hosting guide](https://blog.jonthan.xyz/self-hosting-guide/), but I'm going to sum it up here.

```bash
# We can delete all stoped container, every network not use by al least one container, all dangling images and all dangling build cache.
docker system prune
# We can delete all stoped container, every network not use by al least one container, every unused image and all build cache.
docker system prune -a
# Do the same but filtering specific images
docker system prune --filter nginx:alpine
# Or just the volumes
docker system prune --volumes # First way
docker volume prune # Second way
# Or just the networks
docker volume prune
# Or we can do the same but without asking for confirmation
docker system prune -a -f  
```

That's it, nice and fast.

## Credits

[https://wise.wtf/posts/docker-cleaning/](https://wise.wtf/posts/docker-cleaning/)