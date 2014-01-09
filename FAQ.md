## How do I add a question to this list?

Simply edit this page and add your question to the bottom in the following format:

## How do you pronounce Resque?

/ˈreskyo͞o/

## How do you perform the same job on all connected workers?

For example, perform a disk space check on every machine in a cluster, every machine will run a worker.

## My workers are getting stuck in a weird state.

Are you using **newrelic_rpm**?
This is a known bug: <https://github.com/resque/resque/issues/180>

Whatever the root cause, https://github.com/shaiguitar/resque_stuck_queue/ can help you identify Resque issues coming up before it's too late.

## What's the best way to restart workers using capistrano?

### With Foreman and Upstart

This may not be the _best_ way, but it's an option.  You can manage resque, and any other services your app runs, with foreman; by using upstart or init scripts on the remote server.  See "[Managing and monitoring your Ruby application with Foreman and Upstart](http://michaelvanrooijen.com/articles/2011/06/08-managing-and-monitoring-your-ruby-application-with-foreman-and-upstart/)" for a guide on getting Foreman running.  Once that's done, here's a sample cap task to manage your processes:

```
namespace :foreman do
  desc "Start the application services"
  task :start, :roles => :app do
    sudo "start #{application}"
  end

  desc "Stop the application services"
  task :stop, :roles => :app do
    sudo "stop #{application}"
  end

  desc "Restart the application services"
  task :restart, :roles => :app do
    run "sudo start #{application} || sudo restart #{application}"
  end

  desc "Display logs for a certain process - arg example: PROCESS=web-1"
  task :logs, :roles => :app do
    run "cd #{current_path}/log && cat #{ENV["PROCESS"]}.log"
  end

  desc "Export the Procfile to upstart scripts"
  task :export, :roles => :app do
    # 5 resque workers, 1 resque scheduler
    run "cd #{release_path} && rvmsudo bundle exec foreman export upstart /etc/init -a #{application} -u #{user} -l #{shared_path}/log  -f #{release_path}/Procfile.production -c worker=5,scheduler=1"
  end 
end
```

### Graceful Shutdown

When you write a restart task for resque, you may want to think what happens to a job running at that time. If you want to make sure the job finishes properly, you must send the `QUIT` signal to the existing resque worker process. This will stop the worker process gracefully.

Check out [Readme](https://github.com/resque/resque#workers) and [1-x-stable Readme](https://github.com/resque/resque/blob/1-x-stable/README.markdown#signals) to learn more about how Resque workers respond to signals.

## How do I ensure my Rails classes/environment is loaded?

Make sure you start your workers by specifying `environment` as part of your rake execution.

For example:

```
QUEUE=* rake environment resque:work
```

Some users also experience issues where Rails models are lazy-loaded in the Job execution. Due to some internal class loading wackiness, this results in `LoadErrors` or `Uninitialized Constant` exceptions. This can be solved by forcing ActiveRecord models to load, by creating a `lib/tasks/resque.rake`

``` ruby
# load the Rails app all the time
namespace :resque do
  puts "Loading Rails environment for Resque"
  task :setup => :environment do
    ActiveRecord::Base.descendants.each { |klass|  klass.columns }
  end
end
```

## How do you make a Resque job wait for an ActiveRecord transaction commit?

The first way is to enqueue a job in a model's `after_commit` hook.
But `after_commit` solves the problem only in the model create and update use cases.

In general, a task should be put into redis only after all transactions are complete, otherwise serious problems may appear. In order to do that use the [ar_after_transaction](https://github.com/grosser/ar_after_transaction) gem, which provides an `after_transaction` hook whenever you need it:

``` ruby
ActiveRecord::Base.after_transaction do
  Resque.enqueue(SyncronizationWithGithubWorker, asset.id)
end 
```


If you want to keep the team free from worrying about transaction isolation levels,
then you should monkey patch `Resque.enqueue` to be performed only after the transaction completes.

``` ruby
require 'ar_after_transaction'
require 'resque'
Resque.class_eval do
  class << self
    alias_method :enqueue_without_transaction, :enqueue
    def enqueue(*args)
      ActiveRecord::Base.after_transaction do
        enqueue_without_transaction(*args)
      end
    end
  end
end
```


## How do I print SQL statements in resque?

Add `ActiveRecord::Base.logger = Logger.new(STDOUT)` to your ruby class.

## How do I override views when mounted in a rails app?

No answer.

## Why might Resque be serving up completely blank asset files (CSS & JS) when used with Rails 3.1? 

No answer.

## How do you work around the `MySQL server has gone away` error ?

In your `perform` method, add the following line:

``` ruby
class MyTask
  def self.perform
    ActiveRecord::Base.verify_active_connections!
    # rest of your code
  end
end
```

The Rails doc says the following about `verify_active_connections!`:

    Verify active connections and remove and disconnect connections associated with stale threads.

## How do you use redis' password?

Add the password to your `Redis.new`.

``` ruby
Resque.redis = Redis.new(:host => 'foo',
                         :port => 'foo',
                         :password => 'foo')
```
## How do you protect resque-web with Devise?
Change your `routes.rb` file to authenticate your users.

```ruby
#routes.rb
...
  devise_for :admin_users, ActiveAdmin::Devise.config
  authenticate :admin_user do #replace admin_user(s) with whatever model your users are stored in.
    mount Resque::Server.new, :at => "/jobs"
  end
...
```

If you need to be more specific with your authentication conditions, you can include them in a `:constraints` block when mounting via `match` method.

```ruby
match "/jobs" => Resque::Server, :anchor => false, :constraints => lambda { |req|
    req.env['warden'].authenticated? and req.env['warden'].user.can_view_resque?
  }
```

## How does Resque workers choose which jobs to take from multiple queues?

Check out [lib/resque/worker.rb](https://github.com/resque/resque/blob/master/lib/resque/worker.rb). 

When a worker is initialized it's passed a list of queues, which is checks in the order that they're passed.