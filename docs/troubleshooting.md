# Docker

## Layers folder consuming too much disk space

If you see there is few disk space available may the root cause is that the 
docker layers folder is consuming two much space as described [here](https://forums.docker.com/t/some-way-to-clean-up-identify-contents-of-var-lib-docker-overlay/30604/33).

To understand if that is really the problem you can use the command
`du -h --max-depth=1.` which shows how much disk space each folder is consuming.

If that is the problem you may have to run command `docker system prune -a` that
will prune and remove left overs.

## DNS not resolving

If you are facing DNS issues try to follow the suggestion [here](https://unix.stackexchange.com/a/128223).
