The items that have a strikethrough are currently under development.

The Resque::Failure API presently suffers from a lot of problems. Here are some of the major ones:

* **Too many singletons**: Have a look through [failure.rb](https://github.com/resque/resque/blob/master/lib/resque/failure.rb) and [failure/base.rb](https://github.com/resque/resque/blob/master/lib/resque/failure/base.rb) and you will notice that the API is almost exclusively singletons. There are hardly any instance methods (`base.rb` defines a `#save` method and that's really it)
* ~~**Failures aren't objects**: There's a clear reason why there are so many singletons: when you ask for a failure, what you get back isn't a Resque::Failure object. It's just a JSON blob parsed into a Ruby hash. Want to know anything about the failure? You have to dig through the JSON structure, which depending on where it originated from (lots of people like implementing Resque-alikes in other languages) may or may not have the elements you care about. IMO, when you ask for a failure, you should get a Resque::Failure object.~~
* **Failures don't have unique IDs**: I think that every failure should be identified by a unique ID (similar to a primary key) and that all APIs should accept these IDs. I have done some work with the existing code to try to facilitate this, but the API isn't great.
* ~~**Too many arguments**: Looking at [failure.rb#79](https://github.com/resque/resque/blob/1-x-stable/lib/resque/failure.rb#L79), `Resque::Failure#each` already takes 5 arguments. This seems like a pretty clear candidate for an options hash.~~
* ~~**Wacky APIs**: For whatever reason, by default [Resque::Failure.all](https://github.com/resque/resque/blob/1-x-stable/lib/resque/failure.rb#L79) only returns one failure object (as a parsed JSON blob). To actually return all failures, we need to consult `Resque::Failure.count` first, then pass that into `Resque::Failure#all`. It seems pretty clear to me this API is in need of some love.~~
* **Multiple backends with inconsistent logic**: To see how painful this is, check out [resque-web's RetryController](https://github.com/resque/resque-web/blob/master/app/controllers/retry_controller.rb). ZOMG! That is some gnarly code, working around the above mentioned deficiencies and inconsistencies between the existing failure backends. Ideally, all backends would be unified under a consistent API. That was the original goal of the `Resque::Failure` module, but unfortunately it falls quite short.

### The Path Forward

These are [@tarcieri's](https://github.com/tarcieri) suggestions for fixing the API:

* ~~**More like ActiveRecord**: I think an ActiveRecord-alike API would make `Resque::Failure` intuitive to anyone who's familiar with ActiveRecord, and provides a fairly decent pattern for how these sort of persistence APIs should work. Calling `Resque::Failure.all(:queue => 'myqueue')` should return an array of Resque::Failure objects. I should be able to do things to a `Resque::Failure` object via instance methods, like `#destroy` it or `#retry` it.~~
* **Multiple failure queues or GTFO**: Resque originally supported a single 'failed' queue, unfortunately as more people started using it, this was quickly shown not to scale very well. The failure API should be designed around the idea that every queue has its own individual failure queue. IMO, `Resque::Failure::Redis` should be completely deleted from Resque 2.0, and the `Resque::Failure::RedisMultiQueue` backend should replace it.
* **Support ActiveRecord as a failure backend**: Redis queues are just not a great way to store failures. If one person views the failure page and deletes a failure, and someone else tries to delete a different failure, the second person will end up deleting the wrong failure, because its position in the failure queue has changed. THIS IS BAD! I'm not saying that we should dump a Redis-based failure backend, but it's quite clear that the failure system has outgrown what Redis can do, and really needs a database that can do better querying of the data and can represent each failure with a primary key. I think it would really make sense to (optionally) support an ActiveRecord-based failure backend which provides a lot more rigorous semantics around how failures are handled.

### More Thoughts

These are [@avnerner's](https://github.com/AvnerCohen) thoughts with regards to the new API, less in terms of design and more from client's perspective: 

#### It should be possible to:

* View Failures in the queue sorted as LIFO or FIFO.
* Filter Failures by the (newly suggested) unique identifier
* Filter Failures by class, payload content, whatever, using a provided block

_Above might suggest that the `Resque::Failure` API should be some variation if not exact match of the Enumarable API_

This way, it might be possible to do things like Resque::Failure.any?, Resque::Failure.count and Resque::Failure.reverse.each {.. } 

* It should be possible to requeue Failures, whether it be re-process them or transfer to another queue.
* Failures should have an internal count of retries, as in other queueing systems.
