https://github.com/resque/resque/issues/552

* id based on payload? (resque-meta does it this way)
* Could failures use the same ids? Would it be confusing if failures and metadata objects had different ids?
* It would be nice if you could get the metadata from a failure easily.

* can we just pull resque-meta into core? does it fit everyone's needs?
https://github.com/lmarlow/resque-meta