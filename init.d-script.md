# resque-workers

## Ideas

* Support spawning `resque:workers` for multiple apps.
* Support loading custom environment variables from a config file within each app.
* Support tracking pids via `/var/run/resque/$app-$N.pid` files.
* Automatically detect presence of a `Gemfile.lock` and invoke `bundle exec`.
* Target Ubuntu 12 LTS at first.

## Prototype

A prototype Ubuntu init.d script for resque can be found in [postmodern fork](https://github.com/postmodern/resque/tree/init.d) [here](https://github.com/postmodern/resque/blob/init.d/examples/ubuntu/init.d/resque). This script is currently running in development on a Ubuntu 12.04 LTS VM.

## Existing solutions

* https://gist.github.com/1724049
* http://pastebin.com/UXM15uVy
* http://www.jigsawboys.com/2011/05/25/running-resque-workers-from-upstart-rvm/