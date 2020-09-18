**What's the deal with _ARG_,_FROM_ and _RUN_ in a Dockerfile?**

**ARG**

The only instruction that may precede `FROM` in the Dockerfile.

`FROM` supports variables that are declared by any `ARG` instruction that occurs before the `first FROM`.

**Scenario 1 - How ARG works?**

Dockerfile should look like:

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

Result is the value we assigned to variable `VERSION` which we then echoed to image_version file.

> trusty

An `ARG` declared before a `FROM` is _outside of a build stage_, so it _can’t be used in any instruction after a `FROM`_ **UNLESS** we use an ARG instruction without a value inside of a build stage which in our case is the 2nd declaration of `ARG` in our Dockerfile.



**FROM**

`FROM` instruction initializes a new build stage and sets the `Base Image` for subsequent instructions.

A valid `Dockerfile` must start with a `FROM` instruction (ARG can be used before FROM -- see section `ARG` for explanation) and you can pull the image from

either a Private or Public repository like Docker Hub.

_As a rule of thumb:_

**Use specific FROM `tags` to define what `image version` you're gonna use.**

**You DON’T deploy random versions of your code so why do you deploy random versions OF DEPENDENCIES?**

**Multiple FROM instructions in a Dockerfile**

It is possible for the `FROM` to appear multiple times within a single Dockerfile to create multiple images or `use one build stage as a dependency for 

another`.

**TIP**

Name your build stage with `AS name` added to `FROM` so you can reference it later in the Dockerfile with other `FROM` or `COPY --from=<name>` instructions.

**Scenario 2 - How FROM works?**

Dockerfile should like:

> FROM ubuntu:trusty AS stage1

> RUN cat /proc/version > image_version

> RUN echo "This is stage1" >> image_version

Build the image

> docker build -t from:demo

Run a container and check the contents of the `image_version` file.

> docker run -it from:demo

> root@58aaa5577ee1:/# cat /image_version

> Linux version 5.4.0-47-generic (buildd@lcy01-amd64-014) (gcc version 9.3.0 (Ubuntu 9.3.0-10ubuntu2)) #51-Ubuntu SMP Fri Sep 4 19:50:52 UTC 2020

>This is stage1



**Scenario 3 - How does FROM multi stage works?**

Dockerfile should like:

> FROM ubuntu:trusty AS stage1

> RUN cat /proc/version > image_version

> RUN echo "This is stage1" >> image_version

> FROM alpine:3.12

> COPY --from=stage1 /image_version .

**Explanations and instructions**

We are using the `base image` of `ubuntu:trusty` and we name our build stage `AS stage1` so we can 

reference the build stage in the subsequent `FROM` instruction.

Now in the 2nd stage we are using a much smaller image `FROM alpine:3.12` to which we will copy the `image_version` file.

Using the `COPY` command and `--from=stage1` flag we specify the name of stage1 and then we specify the path where this file can be found 

`/image_version` in our stage1 and we copy it to our current `.` directory in stage2 which is root directory `/`.

Build the image (name it `demo2` or something different than what you named your image in the Scenario 2)

> docker build -t from:demo2 .

Run a container based on the image, once you are in the container check the contents of the /image_version file.

> docker run -it from:demo2

> / # cat /image_version

> Linux version 5.4.0-47-generic (buildd@lcy01-amd64-014) (gcc version 9.3.0 (Ubuntu 9.3.0-10ubuntu2)) #51-Ubuntu SMP Fri Sep 4 19:50:52 UTC 2020

> This is stage1

> / # ls -la | grep image_version

> -rw-r--r--    1 root     root           144 Sep 18 13:50 image_version

As you can see file was copied to our `from:demo2` image from which we launched a container. The image is much smaller in size compared to our **Scenario 2** image `from:demo`.

**Check the size of the images `docker images`**

> REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE

> from                        demo2               f011560b0324        6 minutes ago       5.57MB

> from                        demo                634cde703a8f        9 minutes ago       197MB


#WORK IN PROGRESS#