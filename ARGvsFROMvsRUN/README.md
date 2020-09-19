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



**Scenario 4 - What about the RUN instruction?**

RUN has 2 forms 

> exec form: RUN _["executable", "param1", "param2"]_

> shell form: RUN _command_ the command is run in a shell _/bin/sh -c_ on Linux and _cmd /S /C_ on Windows.

RUN instruction will execute commands in a new layer on top of the image used and commit the results. The resulting image is used for the next step in the `Dockerfile`.

Let's take a quick look at our `Dockerfile` example to better understand how layering works. Each instruction below is creating a layer.

To better explore the layers of an image you can use this awesome tool called [Dive](https://github.com/wagoodman/dive) or you can also explore the layers on image by running `docker image inspect <nameoftheimage>` and at the bottom under `RootFS` you will see section `Layers`

Dockerfile should like:

> FROM ubuntu:trusty AS stage1

> RUN cat /proc/version > image_version

> RUN echo "This is stage1" >> image_version 


The first instruction alone creates a 3xlayers from the ubuntu:trusty Docker image.

The second instruction uses concatenate to print the contents of /proc/version file and pushes this value to a new file called image_version. This creates another layer.

The third instruction uses echo command to append some text at the bottom of the already created image_version file. This creates another layer. Our image will end up with 5 layers. We could reduce the number of layers by modifying the Dockerfile to

> FROM ubuntu:trusty AS stage1

> RUN cat /proc/version > image_version && echo "This is stage1" >> image_version 

Our image will end up with 4 layers now. Layering RUN instructions and generating commits conforms to the core concepts of Docker where commits are cheap and containers can be created from any point in an image’s history.

The `exec` form of the `RUN` instruction allows to run commands using a base image that does not contain the specified shell executable.

To use a different shell, other than ‘/bin/sh’, use the exec form passing in the desired shell.Unlike the shell form, the exec form does not invoke a command shell. This means that normal shell processing does not happen.

> RUN ["/bin/bash","-c","echo Hello"]

In the `shell` form you can use a `\ (backslash)` to continue a single RUN instruction onto the next line. For example:

> RUN cat /proc/version > image_version && `\`
echo "This is stage1" >> image_version

This is equivalent to 

> RUN cat /proc/version > image_version && echo "This is stage1" >> image_version




