# Pull the base image
FROM ubuntu:trusty

# Scenario 1 - CMD shell form
CMD ping localhost

# Scenario 2 - CMD exec form (preffered form)
#CMD ["/bin/ping","localhost"]

# Scenario 3 - CMD as default parameters to ENTRYPOINT in exec form
#ENTRYPOINT ["/bin/ping","-c","5"]
#CMD ["localhost"]

# Scenario 4 - ENTRYPOINT in shell form
#ENTRYPOINT exec ping -c 5 localhost
