---
title: Docker-Compose Best Practices
published: 2022-09-18
description: Just a few to manage a small cluster.
tags: [Docker]
category: Docker
draft: false
---
```bash
# Creates a cluster manager on the current machine.
# Generates a token that we must execute in a worker machine to join it to the cluster.
docker swarm init 
# Gives us the entry token to the cluster for both workers and masters.
docker swarm join-token {worker | master}
# List the nodes.
docker node ls
# The machine leaves the cluster with this command.
docker swarm leave
# Delete the node  
docker node rm
# It creates the service with the flags you indicate and the image you set.
docker service create --name <name> --replicas <replicanumber> --mode global --publish --mode=host published=<externalport>,target=<internalport> --constraint node.labels.env=dev --update-delay <number>s <image>
# List your services.
docker service ls
# List your service instances.
docker service ps <servicename>
# Inspect a service.
docker service --pretty inspect <servicename>
# Stop a service.
docker stop <containerid>
# Scale the service.
docker service scale <nombre o id del servicio>=<número de réplicas>
# Show the logs.
docker logs <servicename or containerID>
# Update the image of a service.
docker service update --image <image>:<version> <servicename>
# Update node info.
docker node update
# Add a label to a node.
docker node update --label-add env=dev
# Undo the last thing you did.
docker service rollback
# Pauses or disables the node so that it stops receiving or updating traffic.
# This way is easier to perform maintenance or remove the node from the swarm.
docker node update --availability drain
```