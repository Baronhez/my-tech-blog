---
title: Docker-Compose Best Practices
published: 2022-08-31
description: Or "The best way to use Docker-Compose", pick your poison.
tags: [Server, Docker]
category: Docker
draft: false
---
Ok, I get it. You want to use Docker-Compose in your working environment, but you don't want to destroy everything in the process, and at the same time, get the most out of it.

Here you go, 6 easy-to-follow tips for you, fellow DevOps!

## Tip #1: Do NOT rebuild your image each time you make a change in code, use volumes instead.

One of my best friends was having a hard time a few months ago, because every time he had to change a line in his python program, he had to rebuild the docker image, and then deploy that new image to the server (what 0 CI/CD pipelines do to a man, hehe).

The answer to that problem was to use volumes from the beginning. If you, wise reader, save the code in a directory and then, when building the image, use that directory as the volume, the only thing you will have to do to apply the changes made to the code is to restart the container.

The most easy-to-see example is a nginx image. Save your .html page in a directory, mount that directory as a volume, like:

```
 volumes:
      - /home/example/index.html:/usr/share/nginx/html:ro
```

And then, build and deploy that image as a container.

If you want to make a change to your web page, change the index.html file and then restart the container. You will see the changes immediately. 

## Tip #2: Be smart, do NOT write the same file two times for dev and prod.

Ok, assuming that you are developing and application, and you need to use some kind of [Webpack](https://webpack.js.org/) or a PostgresDB ONLY in the dev environment... well, you can use the same docker-compose.yml file and the same .env file, but making a lot of changes each time you have to deploy to prod... On the other hand, that's annoying and awful, so, let's use an override file!

But what is an override file and why is it so useful in this case?

Well, imagine that you have a file like this one:

```
# docker-compose.yml
web:
  image: example/my_web_app:latest
  depends_on:
    - db
    - cache

db:
  image: postgres:latest

cache:
  image: redis:latest
```

If you, for testing purposes, want to deploy this application while exposing some ports, mount your code as a volume, and build the web image, then you should use this override file:

```
# docker-compose.override.yml
web:
  build: .
  volumes:
    - '.:/code'
  ports:
    - 8883:80
  environment:
    DEBUG: 'true'

db:
  command: '-d'
  ports:
    - 5432:5432

cache:
  ports:
    - 6379:6379
```

Then, while having both files in the same directory, run:

```
docker-compose up
```

Now, watch in amazement as the services of both archives are deployed at the same time. The reason is that docker-compose search in the current directory for a file named **docker-compose.yml** or **docker-compose.yaml**. Then, search for a file name **docker-compose.override.yml**. Finally, deploys the services contained in both files.

This is really useful, because if you want to not expose those ports in prod, it is as simple as renaming the override file with any other name. By doing this, we will be deploying only the services in the **docker-compose.yml** file. 

You can use this in many ways. For instance, you can have a docker-compose.override.example.yml which you can change and rename as **docker-compose.override.yml** everytime you need to test something in dev. It is similar to when we use a **.env-example** file. In the **.env-example** file we have some default or null values which we can change when creating a **.env** file.

Another use for the override file is when we are deploying an application in some cloud provider, such as AWS or GCP, which provides some service, as postgres management. If we want to use our own postgres in dev environment, we can use an **docker-compose-override.yml** file with the postgres service, while in prod, we only use the **docker-compose.yml**.

Another way to get modularity is to use the command line:

```
# service.yml
services:
  service:
    image: my-image:latest
```

```
# service-dev.yml
services:
  service:
    environment:
      - DEV_MODE=true
```

```
docker-compose -f service.yml -f service-dev.yml up -d
```

##### Credits for this part:

[https://www.youtube.com/watch?v=jGePPQFArwo](https://www.youtube.com/watch?v=jGePPQFArwo)

[https://docs.docker.com/compose/extends/](https://docs.docker.com/compose/extends/)

[https://www.howtogeek.com/devops/how-to-simplify-docker-compose-files-with-yaml-anchors-and-extensions/](https://www.howtogeek.com/devops/how-to-simplify-docker-compose-files-with-yaml-anchors-and-extensions/)

## Tip #3: Be lazy, do NOT rewrite the same yaml parts over and over.

Use YAML Archors for your yaml files if you are using over and over again the same yaml names or steps.

There are 2 parts to this:

-   The anchor '&' which defines a chunk of configuration.  
    
-   The alias '\*' used to refer to that chunk elsewhere.  
    

```
services:
  httpd:
    image: httpd:latest
    restart: &restartpolicy unless-stopped
  mysql:
    image: mysql:latest
    restart: *restartpolicy
```

You can reuse multiple lines:

```
services:
  first:
    image: my-image:latest
    environment: &env
      - CONFIG_KEY
      - EXAMPLE_KEY
      - DEMO_VAR
  second:
    image: another-image:latest
    environment: *env
```

Or even extend those chunks of configuration:

```
services:
  image: my-image:latest
    environment: &env
      - CONFIG_KEY
      - EXAMPLE_KEY
      - DEMO_VAR      
  second:
    image: another-image:latest
    environment:
      <<: *env
      - AN_EXTRA_KEY
      - SECOND_SPECIFIC_KEY  
```

You can use Yaml Archors with Extension fields, because yaml parser will ignore extension fields prefixed with x-. This way, you can reuse, as I mentioned before, chunks of configuration to share configuration:

```
x-env: &env
  environment:
    - CONFIG_KEY
    - EXAMPLE_KEY
 
services:
  first:
    <<: *env
    image: my-image:latest
  second:
    <<: *env
    image: another-image:latest  
```

YAML anchors and aliases cannot contain the ' \[ ', ' \] ', ' { ', ' } ', and ' , ' characters.

Here is a [guide](https://www.howtogeek.com/devops/how-to-simplify-docker-compose-files-with-yaml-anchors-and-extensions/), which is the one I used to write this tip.

## Tip #4: Do NOT forget to refresh environment variables.

Set the restart behavior to restart: always and configure your services with update\_config: true. This way, your environment variables will be refreshed each run.

## Tip #5: Do NOT use "docker rm -f" when cleaning up docker images.

Use this instead:

```
docker rm -f --remove-orphans
```

Regardless of whether we or a running container utilizes them, the "--remove-ophans" flag guarantees that Docker Compose only removes containers and images that are no longer in use.

## Tip #6: Do NOT let your containers consume as much memory and CPU as they want.

Although it may seem unusual and silly, you shouldn't allow any container use all the resources it needs.

You can do this by setting limits:

```
# docker-compose.yml
web:
  deploy:
    resources:
      limits:
        cpus: "1"
```

Before setting these limits, you should know how many resources your services may need to function properly.

And those were the best practices I wanted to share with you, fellow DevOps folks. I hope you find them useful.

## Credits

[https://prod.releasehub.com/blog/6-docker-compose-best-practices-for-dev-and-prod](https://prod.releasehub.com/blog/6-docker-compose-best-practices-for-dev-and-prod)