# Scenario 1 - How ARG works?

# Assign the value `trusty` to variable `VERSION`
#ARG VERSION=trusty

# Pass the variable to FROM instruction image `ubuntu:trusty`
#FROM ubuntu:$VERSION

# Use the default ARG value `trusty` for the second time. Notice we do not assign another value.
#ARG VERSION

# Pass the variable to `echo` command and add the result to a file named `image_version` found at the root of the container.
#RUN echo $VERSION > image_version

# Scenario 2 - How FROM works?
#Pull the base image and name it using `AS name` to reference the build stage with other `FROM` instructions.
FROM ubuntu:trusty AS stage1
# RUN cat command on /proc/version to display the version and put the value into the file `image_version` in the root dir.
RUN cat /proc/version > image_version

FROM ubuntu:trusty
#FROM alpine:3.12
COPY --from=stage1 /image_version .





