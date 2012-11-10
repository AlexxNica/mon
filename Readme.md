
# mon(1)

  Super-simple monitoring program.

  Effectively all it does is daemonize some programs,
  re-executing the program on failure, write pidfiles,
  and provide status checks.

## Installation

```
$ make install
```

## Usage

```

Usage: mon [options] <command>

Options:

  -V, --version                 output program version
  -h, --help                    output help information
  -l, --log <path>              specify logfile [mon.log]
  -s, --sleep <sec>             sleep seconds before re-executing [1]
  -S, --status                  check status of --pidfile
  -p, --pidfile <path>          write pid to <path>
  -m, --mon-pidfile <path>      write mon(1) pid to <path>
  -P, --prefix <str>            add a log prefix
  -d, --daemonize               daemonize the program
  -a, --attempts <n>            retry attempts within 60 seconds [10]
  -R, --on-restart <cmd>        execute <cmd> on restarts
  -E, --on-error <cmd>          execute <cmd> on error

```

## Example

  The most simple use of `mon(1)` is to simply keep a command running:

```
$ mon ./myprogram
mon : pid 50395
mon : child 50396
mon : sh -c "./example/program.sh"
one
two
three
```

  You may daemonize mon and disassociate from the term with `-d`:

```
$ mon ./myprogram -d
mon : pid 50413
```

## Groups of mon

  `mon(1)` is designed to monitor a single program only, this means a few things,
  firstly that a single `mon(1)` may crash and it will not influence other programs,
  secondly that the "configuration" for `mon(1)` is simply a shell script,
  no need for funky weird inflexible DSLs.

```bash
#!/usr/bin/env bash

pids="/var/run"
app="/www/example.com"

mon -d redis-server -p $pids/redis.pid
mon -d "node $app/app" -p $pids/app-0.pid
mon -d "node $app/jobs" -p $pids/jobs-0.pid
mon -d "node $app/jobs" -p $pids/jobs-1.pid
mon -d "node $app/jobs" -p $pids/jobs-2.pid
mon -d "node $app/image" -p $pids/image-0.pid
mon -d "node $app/image" -p $pids/image-1.pid
mon -d "node $app/image-broker" -p $pids/image-broker.pid
```

## Restarting processes

  Restarting a process is simple, `kill(1)` it. For example if your app is written
  to gracefully exit on __SIGQUIT__ you might want to do:
  
    $ kill -s SIGQUIT $(cat /var/run/app-0.pid)

  Or if you just want things done quick and dirty:

    $ kill -s SIGKILL $(cat /var/run/app-0.pid)

  `mon(1)` will see that the child died, and re-execute
  the initial start command.

## Links

  Tools built with `mon(1)`:
  
  - [mongroup(1)](https://github.com/jgallen23/mongroup) - monitor a group of processes