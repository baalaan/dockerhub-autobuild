**dockerhub-autobuild**

I created this repo as a learning experience for Git and Docker and how the tools can interact to each other.

On Docker Hub I have create a Public repo called [stefandockerid/autobuilds](https://hub.docker.com/r/stefandockerid/autobuilds) which has the **Automated builds** feature enabled which essentially triggers a new docker image build every time I `git push` a Dockerfile to certain code repository folders.


**CMDvsENTRYPOINT**
Contains a Dockerfile with a very basic Ubuntu image and with the help of the README.md instructions guides the user to undestand the difference between using 

> `CMD` and/or `ENTRYPOINT` in `shell` or `exec` form in a Dockerfile.

You are encouraged to experiment with your own variations!

Good luck!