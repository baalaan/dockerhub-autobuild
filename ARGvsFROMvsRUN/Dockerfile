# Scenario 1 - How ARG works?
# Assign the value `trusty` to variable `VERSION`
ARG VERSION=trusty
# Pass the variable to FROM instruction image `ubuntu:trusty`
FROM ubuntu:$VERSION
# Use the default ARG value `trusty` for the second time. Notice we do not assign another value.
ARG VERSION
# Pass the variable to `echo` command and add the result to a file named `image_version` found at the root of the container.
RUN echo $VERSION > image_version


