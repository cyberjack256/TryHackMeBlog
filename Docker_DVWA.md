
---
title:  Docker_DVWA.md
description: Setup and Configuration of Docker DVWA
service: Docker, DVWA
author: Jack Turner
---

# Docker Damn Vulnerable Web Application (DVWA) Setup and Configuration

### Index

- [Downloading](#downloading)

- [Running](#running)

- [Access](#access)

- [Configuration](#configuration)

- [Persistence](#persistence)

- [Additional Resources](#additional_resources)


#### Downloading

###   ⚠**Disclaimer**⚠

> Damn Vulnerable Web Application is damn vulnerable! **Do not** upload it to your hosting provider's public html folder or any Internet facing servers, as they will be **compromised**. 
> We **do not** take responsibility for the way in which any one uses this application (DVWA). We have made the purposes of the application clear and it **should not** be used maliciously. We have given warnings and taken measures to prevent users from installing DVWA on to live web servers. If your web server is compromised via an installation of DVWA, it is not our responsibility, it is the responsibility of the person/s who uploaded and installed it. // Quoted from DVWA page, formatted by me

Pull the docker image as described in the [Docker section](https://github.com/digininja/DVWA#docker-container) of GitHub README.

```sh
# DockerHub: https://hub.docker.com/r/vulnerables/web-dvwa/

$ docker pull vulnerables/web-dvwa
```

Ensure docker image is available

```sh
$ docker images

REPOSITORY             TAG       IMAGE ID       CREATED        SIZE
vulnerables/web-dvwa   latest    ab0d83586b6e   3 years ago    712MB
```


#### Running

Run the docker image using the following options

`docker run --rm -it -d -p 127.0.0.1:8080:8080 vulnerables/web-dvwa`

```text
--rm                   Automatically remove the container when it exits
-i, --interactive      Keep STDIN open even if not attached
-t, --tty              Allocate a pseudo-TTY
-d, --detach           Run container in background and print container ID
-p, --publish list     Publish a container's port(s) to the host
```

Using `-p 127.0.0.1:8080:8080` will ensure that docker is not exposing the port to the outside. It binds `:80` for `127.0.0.1` interface only.


#### Access

If your DVWA is also bound to `:8080`, navigate in browser to `localhost`.


Login using default `admin/password` credentials.


As you can see, my instance is missing the following features:
* PHP allow_url_include
* reCAPTCHA key

#### Configuration

Copy `php.ini` directly from the docker using `docker cp` command and change the turn `allow_url_include` on

```sh
$ docker ps
CONTAINER ID   IMAGE                  COMMAND      CREATED          STATUS          PORTS                  NAMES
6ba0d3c87c1a   vulnerables/web-dvwa   "/main.sh"   16 minutes ago   Up 16 minutes   127.0.0.1:8080->8080/tcp   friendly_mcnulty

$ docker cp 6ba0d3c87c1a:/etc/php/7.0/apache2/php.ini .
$ subl php.ini
```

Copy the `php.ini` back to docker container

```sh
$ docker cp php.ini 6ba0d3c87c1a:/etc/php/7.0/apache2/php.ini
```

Attach to the docker instance with bash shell and restart Apache

```text
$ docker exec -it 6ba0d3c87c1a bash
root@6ba0d3c87c1a:/# service apache2 restart
[....] Restarting Apache httpd web server: apache2AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
. ok
```

Now if you refresh the page you will see that `allow_url_include` is no longer reporting as disabled.


##### reCAPTCHA

Now to the reCAPTCHA configuration. Go to [reCAPTCHA Admin](https://google.com/recaptcha/admin/create) (or search for "google recaptcha admin"). Generate v3 API keys, domain names don't matter.


You can navigate in docker shell to `/var/www/html/config` and try to modify the config file - but none of the text editors is installed, so we go with the `docker cp` way.

Add reCAPTCHA keys to the config file:

```sh
$ docker cp 6ba0d3c87c1a:/var/www/html/config/config.inc.php .
$ subl config.inc.php
```


And copy back to the docker.

`$ docker cp config.inc.php 6ba0d3c87c1a:/var/www/html/config/`

Now last time, refresh the `setup.php`


Click _Create/Reset Database_. You will be logged out after a while, login again. Congratulation, you have configured **Damn Vulnerable Web Application**.


Last thing to say is that DVWA has 4 difficulty (security) settings. Go to the _DVWA Security_ section to set the _Security Level_.


#### Persistence


Now that we have modified the **running** docker container, we should somehow make our changes persist through consecutive runs (because next time we start a docker image it will start as a fresh instance - that's the whole idea of containers). This can be done at least in two ways, I am aware of - and I didn't find any meaningful differences between them.

##### Patch the original image

```sh
$ docker ps
CONTAINER ID   IMAGE                  COMMAND      CREATED       STATUS       PORTS                  NAMES
6ba0d3c87c1a   vulnerables/web-dvwa   "/main.sh"   16 minutes ago   Up 16 minutes   127.0.0.1:8080->8080/tcp   friendly_mcnulty

$ docker commit 6ba0d3c87c1a vulnerables/web-dvwa:patched
```

And verify that a new image is created.

```sh
$ docker images
REPOSITORY                 TAG       IMAGE ID       CREATED          SIZE
vulnerables/web-dvwa       latest    ab0d83586b6e   3 years ago      712MB

$ docker images vulnerables/web-dvwa
REPOSITORY             TAG       IMAGE ID       CREATED          SIZE
vulnerables/web-dvwa   latest    ab0d83586b6e   3 years ago      712MB
```

If you want to run patched version, specify tag, like so:

```sh
$ docker run --rm -it -d -p 127.0.0.1:8080:8080 vulnerables/web-dvwa:patched
```

##### Create a new image

```sh
$ docker ps
CONTAINER ID   IMAGE                  COMMAND      CREATED       STATUS       PORTS                  NAMES
6ba0d3c87c1a   vulnerables/web-dvwa   "/main.sh"   16 minutes ago   Up 16 minutes   127.0.0.1:8080->8080/tcp   friendly_mcnulty

$ docker commit 6ba0d3c87c1a vulnerables/dvwa-patched
```

And verify that a new image is created - you start patched version by running this new image.

```sh
$ docker images
REPOSITORY                 TAG       IMAGE ID       CREATED          SIZE
vulnerables/web-dvwa       latest    ab0d83586b6e   3 years ago      712MB
```


#### Additional_Resources

* [DVWA Home](https://dvwa.co.uk/)
* [Official DockerHub for DVWA](https://hub.docker.com/r/vulnerables/web-dvwa)
* [Docker Docs: Container networking](https://docs.docker.com/config/containers/container-networking/)
* [Publish or expose port (-p, --expose)](https://docs.docker.com/engine/reference/commandline/run/#publish-or-expose-port--p---expose)
* [Google reCAPTCHA Docs](https://developers.google.com/recaptcha/docs/v3)