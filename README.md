docker-exec
===========

Wrapper script around `docker start/stop/restart` that provides basic UNIX
signal handling. Currently handles INT, TERM, and HUP. Also provides log output.

Docker doesn't provide proper signal handling yet, and I badly wanted to have
that in order to supervise containers using [Runit](http://smarden.org/runit/).
In addition to that, I wanted to learn a little more about UNIX signals, and
Bash scripting.

**Since Docker 0.6**, this script is obsolete because interactive mode now
correctly passes SIGINT and SIGHUP to the child process within the container.

With Runit, use the following `run` script.

```sh
#!/bin/bash
exec 2>&1

command='echo "started"; while true; do date; sleep 1; done'
exec docker run -i ubuntu /bin/sh -c "$command"
```


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

With Runit, your `run` script would look like this:

```sh
#!/bin/sh
exec 2>&1

command='echo "started"; while true; do date; sleep 1; done'
cid=$(docker run -d ubuntu /bin/sh -c "$command")

exec docker-exec $cid
```


Known issues
------------

- Output of `docker-exec` will stall if another processes stops the container,
  since it doesn't notice that `docker attach` exits. Workaround: send INT/TERM
  and restart the script, or send HUP in order to restart the container and
  re-attach to the logs.
