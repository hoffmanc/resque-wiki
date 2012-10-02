# resque-workers

## Ideas

* Support spawning `resque:workers` for multiple apps.
* Support loading custom environment variables from a config file within each app.
* Support tracking pids via `/var/run/resque/$app-$N.pid` files.
* Automatically detect presence of a `Gemfile.lock` and invoke `bundle exec`.
* Target Ubuntu 12 LTS at first.

## Existing solutions

* https://gist.github.com/1724049
* http://pastebin.com/UXM15uVy
* http://www.jigsawboys.com/2011/05/25/running-resque-workers-from-upstart-rvm/