Docker Alpine PHP CLI Image
==================================================

A PHP CLI based on Alpine.

```
# Bash into the container and mount the current directory
$ docker run --rm -it -v$PWD:/www meg4r0m/alpine-php-cli bash
```

If you are using the source repository, you can use the provided Runfile for 
some additional convenience commands.

```
$ gem install runfile
$ run --help
```

- [View on Docker Hub][1]
- [View on GitHub][2]

[1]: https://hub.docker.com/r/meg4r0m/alpine-php-cli/
[2]: https://github.com/psykoterro/docker-alpine-php-cli