docker-exec
===========

Wrapper script around `docker start/stop/restart` that provides basic UNIX
signal handling. Currently handles INT, TERM, and HUP. Also provides log output.

Docker doesn't provide proper signal handling yet, and I badly wanted to have
that in order to supervise containers using [Runit](http://smarden.org/runit/).
In addition to that, I wanted to learn a little more about UNIX signals, and
Bash scripting.

Official signal handling support for Docker is in the works:
[dotcloud/docker#1249](https://github.com/dotcloud/docker/pull/1249)


Usage
-----

*Hint: You can simply paste this code into your Bash session.*

```sh
# run a command that periodically prints
command='echo "started"; while true; do date; sleep 1; done'
cid=$(docker run -d ubuntu /bin/sh -c "$command")

# start the wrapper and redirect its output (actually: the command's output)
./docker-exec $cid > current.log &

# stop the container (waits for 5 seconds before killing it)
kill -TERM $!
```

`docker-exec` will return with the container command's exit code.


Known issues
------------

- Output of `docker-exec` will stall if another processes stops the container,
  since it doesn't notice that `docker attach` exits. Workaround: send INT/TERM
  and restart the script, or send HUP in order to restart the container and
  re-attach to the logs.
