## How do I add a question to this list?

Simply edit this page and add your question to the bottom in the following format:

``` markdown

## How do you pronounce Resque?

No answer.
```


Someone will (hopefully) fill in the answer.

## My workers are getting stuck in a weird state.

Are you using **newrelic_rpm**? This is a known bug: <https://github.com/defunkt/resque/issues/180>

## What's the best way to restart workers using capistrano?

### With Foreman and Upstart

This may not be the "best" way, but it's an option.  You can manage resque and any of the other services your app runs with foreman (using upstart or init scripts on the remote server).  See "[Managing and monitoring your Ruby application with Foreman and Upstart](http://michaelvanrooijen.com/articles/2011/06/08-managing-and-monitoring-your-ruby-application-with-foreman-and-upstart/)" for a great guide on getting Foreman running.  Once that's done, here's a sample cap task to manage your processes:

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
    run "cd #{release_path} && rvmsudo bundle exec foreman export upstart /etc/init -a #{application} -u #{user} -l #{shared_path}/log  -f #{release_path}/Procfile.production -c worker=5 scheduler=1"
  end 
end
```

### Graceful Shutdown

When you write a restart task for resque, you may want to think what happens to a job running at that time. If you want to make sure the job finishes properly, you better send `QUIT` signal to the existing resque worker process so that the worker process will stop gracefully.

Check out [Readme](https://github.com/defunkt/resque/blob/master/README.markdown) to know more how Resque worker responds signals.

## How do I ensure my Rails classes/environment is loaded?

Make sure you start your workers by specifying ```environment``` as part of your rake execution, e.g. 

```
QUEUE=* rake environment resque:work
```

Some users also see issues where Rails models are lazy-loaded in the Job execution and due to some internal class loading wackiness this results in LoadErrors or Uninitialized Constant exceptions. This can be solved by forcing ActiveRecord models to load, by creating a ```lib/tasks/resque.rake```

``` ruby
# load the Rails app all the time
namespace :resque do
  puts "Loading Rails environment for Resque"
  task :setup => :environment do
    ActiveRecord::Base.descendants.each { |klass|  klass.columns }
  end
end
```

## How to make Resque job to wait for ActiveRecord transaction commit, so that it see all changes made by that transaction?

The first way is enqueue job only in `after_commit` hook in models.
But `after_commit` solves the problem only in model create/update use case . In general task should be put to redis only after all transactions get closed. Otherwise serious problems may appear.

In order to do that use [ar_after_transaction](https://github.com/grosser/ar_after_transaction) gem that provides `after_transaction` hook whenever you need it:

``` ruby
ActiveRecord::Base.after_transaction do
  Resque.enqueue(SyncronizationWithGithubWorker, asset.id)
end 
```


If you want to keep entire team free from knowledge about transaction isolation level and this problem, you should monkey patch `Resque.enqueue` to be performed always after transaction completes.

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

## How do I override the views when mounted in a rails app?

No answer.