Resque ships with Redis and [Airbrake](http://airbrake.io/) failure backends.

## Redis backends

Resque ships with two failure backends which can be controlled by setting the `FAILURE_BACKEND` environment variable (or `Resque.config.failure_backend` on Resque 2.0) to the given string:

* `'redis'`: (default) this creates a single 'failure' queue within Redis in which all failures from all jobs are placed, regardless of which queue the job originated from.
* `'redis_multi_queue'`: creates a separate failure queue per job queue. Especially useful if you have a large number of job queues and want a way to individually track failures on a queue-by-queue basis

## Defining custom failure backends

* **Email**: http://gist.github.com/291329
* **Email via exception_notification**: https://github.com/akshayrawat/resque_exception_notification - Sends data via the [exception_notification](https://github.com/smartinez87/exception_notification) gem for Rails.
* **Rescue log** (often STDOUT): https://github.com/orgsync/resque-backtrace - Primarily for local debugging
* **Syslog**:http://gist.github.com/1779106
* **Custom**:http://gist.github.com/299477
* **Coalmine**:https://github.com/Fatsoma/resque_coalmine_gem - Sends data to [Coalmine](https://www.getcoalmine.com/)
* **Exceptional**: http://github.com/lantins/resque-exceptional - Sends data to [Exceptional](http://www.getexceptional.com/)
* **Noti**: https://github.com/jpl/resque-noti-failure - Sends data to [Noti](https://notiapp.com/)
* **Rollbar**: https://github.com/CrowdFlower/resque-rollbar - Sends data to [Rollbar](https://rollbar.com)
* **Slack**: https://github.com/julienXX/resque-slack - Sends data to a [Slack](https://slack.com) channel

## Using multiple failure backends at once

Using multiple failure backends is also useful. For example, you may want to get an Airbrake notification and be able to view the same exception under the 'Failed' tab in ResqueWeb.  To use multiple failure backends, add something like the following to an initializer, rake task, or whatever:

```ruby
require 'resque/failure/multiple'
require 'resque/failure/airbrake'
require 'resque/failure/redis'
Resque::Failure::Airbrake.configure do |config|
  config.api_key = 'fc49503482dec7bf13eda286c99ab2bf'
  config.secure = true # only set this to true if you are on a paid Airbrake plan
end
Resque::Failure::Multiple.classes = [Resque::Failure::Redis, Resque::Failure::Airbrake]
Resque::Failure.backend = Resque::Failure::Multiple
```