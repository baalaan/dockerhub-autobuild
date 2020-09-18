**What's the deal with _ARG_,_FROM_ and _RUN_ in a Dockerfile?**

**ARG**
The only instruction that may precede `FROM` in the Dockerfile.

`FROM` supports variables that are declared by any `ARG` instruction that occurs before the `first FROM`.

**Scenario 1 - How ARG works?**

Dockerfile should look like

> ARG VERSION=trusty

> FROM ubuntu:$VERSION

> ARG VERSION

> RUN echo $VERSION > image_version

**Explanations and instructions**

Build the image

> docker build -t arg:demo .

Run a container based on the image. (`-it` option brings up an interactive terminal)

> docker run -it arg:demo

Check the file image_version inside the container

> root@4c533ebe3586:/# cat /image_version
> trusty

An `ARG` declared before a `FROM` is _outside of a build stage_, so it _can’t be used in any instruction after a `FROM`_ **UNLESS** we use an ARG instruction without a value inside of a build stage which in our case is the 2nd declaration of `ARG` in our Dockerfile.



**FROM**

`FROM` instruction initializes a new build stage and sets the `Base Image` for subsequent instructions.

A valid `Dockerfile` must start with a `FROM` instruction and you can pull the image from either a Private or Public repository like Docker Hub.

_As a rule of thumb:_
**Use specific FROM `tags` to define what `image version` you're gonna use.**

**You DON’T deploy random versions of your code so why do you deploy random versions OF DEPENDENCIES?**

