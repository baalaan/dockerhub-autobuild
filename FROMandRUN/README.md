##**What's the deal with _FROM_ and _RUN_ in a Dockerfile?**##

##**FROM**##

`FROM` instruction initializes a new build stage and sets the `Base Image` for subsequent instructions.

A valid `Dockerfile` must start with a `FROM` instruction and you can pull the image from either a Private or Public repository like Docker Hub.

_As a rule of thumb:_
**Use specific FROM `tags` to define what `image version` you're gonna use.**

**You DON’T deploy random versions of your code so why do you deploy random versions OF DEPENDENCIES?**

