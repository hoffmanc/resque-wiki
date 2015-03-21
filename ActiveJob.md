Rails 4.2 introduced ActiveJob, a standard interface for job runners. Here is how to use Resque with ActiveJob.

Set `active_job.queue_adapter` to `:resque` in `config/application.rb`.

