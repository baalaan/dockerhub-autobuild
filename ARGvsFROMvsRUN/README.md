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

> FROM alpine:3.12

> COPY --from=stage1 /image_version .

**Explanations and instructions**

We are using the `base image` of `ubuntu:trusty` and we name our build stage `AS stage1` so we can 

reference the build stage in the subsequent `FROM` instruction.

Using the `RUN` command we `cat` the contents of `/proc/version` file to file `image_version` which is 

going to be placed in our root directory.


Now in the 2nd stage we are using a much smaller image `FROM alpine:3.12` to which we will copy the `image_version` file.

Using the `COPY` command and `--from=stage1` flag we specify the name of stage1 and then we specify the path where this file can be found 

`/image_version` in our stage1 and we copy it to our current `.` directory in stage2 which is root directory `/`.
