Resque supports any Logger that is a duck type of the Ruby standard library's built-in [Logger class](http://www.ruby-doc.org/stdlib-1.9.3/libdoc/logger/rdoc/Logger.html). By default Resque will log to STDOUT. You can reconfigure the logger with

```ruby
Resque.logger = MyLogger.new
```