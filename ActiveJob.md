Rails 4.2 introduced ActiveJob, a standard interface for job runners. Here is how to use Resque with ActiveJob.

Set `active_job.queue_adapter` to `:resque` in `config/application.rb`.

Change your job class to inherit from `ActiveJob::Base`:
```ruby
class Archive < ActiveJob::Base
  queue_as :default

  def perform(repo_id, branch = 'master')
    repo = Repository.find(repo_id)
    repo.create_archive(branch)
  end
end
```