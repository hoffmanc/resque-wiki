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
No answer.
## How do I ensure my Rails classes/environment is loaded?

Make sure you start your workers by specifying ```environment``` as part of your rake execution, e.g. 

```
QUEUE=* rake environment resque:work
```

Some users also see issues where Rails models are lazy-loaded in the Job execution and due to some internal class loading wackiness this results in LoadErrors or Uninitialized Constant exceptions. This can be solved by forcing ActiveRecord models to load, by creating a ```lib/tasks/resque.rake```

```
# load the Rails app all the time
namespace :resque do
  puts "Loading Rails environment for Resque"
  task :setup => :environment
  ActiveRecord::Base.send(:descendants).each { |klass|  klass.columns }
end
```