What is the difference between the shell form and exec form of CMD and ENTRYPOINT?

CMD (https://docs.docker.com/engine/reference/builder/#cmd)
There can only be one CMD instruction in a Dockerfile. If you list more than one CMD then only the last CMD will take effect.
The main purpose of a CMD is to provide defaults for an executing container.

You will find multiple scenarios created in the Dockerfile which will help you better understand the behavior of the CMD and ENTRYPOINT in shell and exec form. This means that you will have to comment, uncomment, build and rebuild images, create and recreate containers multiple times to see what is the difference between the scenarios and their outcomes.

The Dockerfile uses image "FROM ubuntu:trusty". You can check the details of the image on Docker Hub: https://hub.docker.com/layers/ubuntu/library/ubuntu/trusty/images/sha256-82c071cd1155f27d6ea9da17a3dd8efa0f461b70b8958184503e00c6e6dd10ef?context=explore 


Most of the images you will find on Docker Hub will use a shell like /bin/sh or /bin/bash

Scenario 1 - CMD shell form
Dockerfile should be like this (all the other lines should be commented with # at the beginning of the row):

FROM ubuntu:trusty
CMD ping localhost

Explanation and instructions
1. CMD in the shell form -- CMD executable param1 param2 
-- e.g CMD ping localhost -- ping -- is the executable -- localhost -- is parameter1
2. When using the shell form, the specified binary is executed with an invocation of the shell using: /bin/sh -c
3. To see how this works you need to build an image based on the Dockerfile by following the instructions below

Change folder to where you saved/created the Dockerfile and run the commands below:

Build the image
docker build -t ubuntu:ping .
-- our image will be named and tagged <name>:<tag> ubuntu:ping 
-- you can check this by running $ docker images

Run a container based on the image
docker container run -dit ubuntu:ping

Because we ran the container in '-d' detached mode you will not see the output but we can use the logs command to see if the ping command is working inside the container.
Check the name of the container: docker ps -a
docker container logs -f <nameofyourcontainer>

You will immediately see the following output which you can stop by pressing 'Ctrl + C' on your keyboard. This means the ping command is running inside the container and it pings the localhost 127.0.0.1
--------------------------------------------------------------------
PING localhost (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.034 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.089 ms
64 bytes from localhost (127.0.0.1): icmp_seq=3 ttl=64 time=0.090 ms
^C
--------------------------------------------------------------------


Check what processes are running inside the container.
docker exec <nameofyourcontainer> ps aux
--------------------------------------------------------------------
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   4452   788 pts/0    Ss+  07:53   0:00 /bin/sh -c ping localhost
root           6  0.0  0.0   8608  1884 pts/0    S+   07:53   0:00 ping localhost
root           7  0.0  0.0  15576  2140 ?        Rs   07:56   0:00 ps aux
--------------------------------------------------------------------

Notice how root PID=1 is /bin/sh executable and not our ping command?
This can be problematic in certain scenarios when you are building a minimal image which does not have a shell binary because when Docker is constructing the command it doesn't check to see if the shell is or is not available in the container so if you don't have /bin/sh in the image, the container will not start. We can solve this problem by using the exec form.
--------------------------------------------------------------------
--------------------------------------------------------------------
--------------------------------------------------------------------
Scenario 2 - CMD exec form
Dockerfile should be like this (all the other lines should be commented with # at the beginning of the row):

FROM ubuntu:trusty
CMD ["/bin/ping","localhost"]

Explanation and instructions
1. CMD in the exec form -- CMD ["executable", "param1", "param2"]
-- e.g CMD ["/bin/ping","localhost"]-- /bin/ping -- is the executable -- localhost -- is parameter1
2. When the exec form of the CMD is used the command will be executed without a shell.
3. Rebuild the image based on the new Dockerfile by following the instructions below

Build the image
docker build -t ubuntu:ping .

Run a container
docker run -d ubuntu:ping

Display the latest container created
docker ps -l

Check what processes are running inside the container.
docker exec <nameofyourcontainer> ps aux
--------------------------------------------------------------------
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   8608  1924 ?        Ss   08:25   0:00 /bin/ping localhost
root           6  0.0  0.0  15576  2120 ?        Rs   08:29   0:00 ps aux
--------------------------------------------------------------------

Notice how root PID=1 is '/bin/ping localhost'? This is because it's run directly without the shell process.
In contrast to previous '/bin/sh -c ping localhost'.

The general recommendation is to use the EXEC form as it's quite obvious what command is running as PID=1 in your container and if you need to send POSIX signals to the container since /bin/sh won't forward signals to child processes because your image might not have /bin/sh.
--------------------------------------------------------------------
--------------------------------------------------------------------
--------------------------------------------------------------------
Scenario 3 - CMD as default parameters to ENTRYPOINT in exec form

FROM ubuntu:trusty
ENTRYPOINT ["/bin/ping","-c","5"]
CMD ["localhost"]

Explanation and instructions
1. CMD as default parameters to ENTRYPOINT -- CMD ["param1", "param2"]
-- e.g CMD ["localhost"]-- localhost -- is parameter1
2. When CMD is used as default parameter to ENTRYPOINT the parameter (localhost) will be passed/appended to the ENTRYPOINT e.g '/bin/ping -c 5 localhost'
3. One thing to keep in mind is that CMD parameter value can be easily overridden by passing arguments to the `docker run` command based on this image.
4. Rebuild the image based on the new Dockerfile by following the instructions below

Build the image
docker build -t ubuntu:ping .

Run a container
docker run ubuntu:ping
-------------------------------------------------------------------
PING localhost (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.026 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.088 ms
64 bytes from localhost (127.0.0.1): icmp_seq=3 ttl=64 time=0.088 ms
64 bytes from localhost (127.0.0.1): icmp_seq=4 ttl=64 time=0.087 ms
64 bytes from localhost (127.0.0.1): icmp_seq=5 ttl=64 time=0.087 ms

--- localhost ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4090ms
rtt min/avg/max/mdev = 0.026/0.075/0.088/0.025 ms
-------------------------------------------------------------------

5. Now let's override the localhost parameter when running the `docker run` command by pinging google.com
docker run ubuntu:ping google.com
-------------------------------------------------------------------
PING google.com (216.58.214.238) 56(84) bytes of data.
64 bytes from suchsecretmuchwow.net (216.58.214.238): icmp_seq=1 ttl=117 time=22.4 ms
64 bytes from suchsecretmuchwow.net (216.58.214.238): icmp_seq=2 ttl=117 time=20.3 ms
64 bytes from suchsecretmuchwow.net (216.58.214.238): icmp_seq=3 ttl=117 time=19.4 ms
64 bytes from suchsecretmuchwow.net (216.58.214.238): icmp_seq=4 ttl=117 time=21.7 ms
64 bytes from suchsecretmuchwow.net (216.58.214.238): icmp_seq=5 ttl=117 time=22.5 ms

--- google.com ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4007ms
rtt min/avg/max/mdev = 19.406/21.296/22.575/1.238 ms
-------------------------------------------------------------------

By passing the argument `google.com` to the `docker run` command we are essentially overriding the `localhost` parameter given by the CMD in the Dockerfile.

Notice how `-c 5` (--count 5) is now part of the default command and it allows only 5 replies? It looks like it's hardcoded.

6. Now let's override the ENTRYPOINT when running the `docker run` command.

Use --entrypoint flag and change the ENTRYPOINT to `--entrypoint /bin/ping` IMAGENAME `-c 3 google.com` 
docker run --entrypoint /bin/ping ubuntu:ping -c 3 google.com
-------------------------------------------------------------------
PING google.com (216.58.214.238) 56(84) bytes of data.
64 bytes from suchsecretmuchwow.net (216.58.214.238): icmp_seq=1 ttl=117 time=21.9 ms
64 bytes from suchsecretmuchwow.net (216.58.214.238): icmp_seq=3 ttl=117 time=23.1 ms

--- google.com ping statistics ---
3 packets transmitted, 2 received, 33% packet loss, time 6052ms
rtt min/avg/max/mdev = 21.976/22.563/23.151/0.606 ms
-------------------------------------------------------------------

Notice how now we did the override as there are only 3 replies instead of 5?

7. Another override the ENTRYPOINT example
docker run --entrypoint /bin/echo ubuntu:ping Hello World
-------------------------------------------------------------------
Hello World
-------------------------------------------------------------------

Notice how now we completely changed the command from `ping` to `echo`?
--------------------------------------------------------------------
--------------------------------------------------------------------
--------------------------------------------------------------------
Scenario 4 - ENTRYPOINT in shell form -- READ MORE: https://docs.docker.com/engine/reference/builder/#understand-how-cmd-and-entrypoint-interact

FROM ubuntu:trusty
ENTRYPOINT exec ping -c 5 localhost


Explanation and instructions
You can specify a plain string for the ENTRYPOINT and it will execute in `/bin/sh -c`. This form will use shell processing to substitute shell environment variables, and will ignore any `CMD` or `docker run` command line arguments. To ensure that docker stop will signal any long running ENTRYPOINT executable correctly, you need to remember to start it with `exec`.

Build the image
docker build -t ubuntu:ping .

Run a container
docker run ubuntu:ping
-------------------------------------------------------------------
PING localhost (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.030 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.086 ms
64 bytes from localhost (127.0.0.1): icmp_seq=3 ttl=64 time=0.090 ms
64 bytes from localhost (127.0.0.1): icmp_seq=4 ttl=64 time=0.088 ms
64 bytes from localhost (127.0.0.1): icmp_seq=5 ttl=64 time=0.088 ms

--- localhost ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4078ms
rtt min/avg/max/mdev = 0.030/0.076/0.090/0.024 ms
-------------------------------------------------------------------

Notice how after 5 replies it stops? This is because we passed the `-c 5` argument to the command.

THINGS TO REMEMBER
------------------
When using ENTRYPOINT and CMD together it's important that you always use the exec form of both instructions.
Trying to use the shell form, or mixing-and-matching the shell and exec forms will almost never give you the result you want.




 




