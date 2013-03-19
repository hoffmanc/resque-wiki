## Style Guide

Please stick to the coding style that already exists in Resque. When in doubt, check the excellent [RUBY-STYLE](http://github.com/chneukirchen/styleguide/blob/master/RUBY-STYLE) document.

## Adding Code

* Document every method you add. ([example][document_example])
* Write a test for every method you add. ([example][test_example])
* If your addition is something you wish you had read in the README, add it to the README.
* Wrap your lines at 80 characters. If your code can't fit, refactor.

[document_example]: http://github.com/defunkt/resque/blob/53c074177c32ba8e8e4896372b6d6fbc5f9efacf/lib/resque.rb#L74-78
[test_example]: http://github.com/defunkt/resque/blob/53c074177c32ba8e8e4896372b6d6fbc5f9efacf/test/resque_test.rb#L135-144

## Asking Questions

* We have a mailing list for discussions. To join the list simply send an email to [resque@librelist.com](mailto:resque@librelist.com). This will subscribe you and send you information about your subscription, including unsubscribe information.

##. Bugs

* The [Issue Tracker][issue_tracker] is the perfect place for bugs.
* Have a large stack trace you want to paste in? Create a [Gist][gist] and link to it in your issue, or, if you're on OS X, try the handy [pbindent][pbindent] command.

[issue_tracker]: http://github.com/defunkt/resque/issues
[gist]: http://gist.github.com/
[pbindent]: http://github.com/rtomayko/dotfiles/blob/rtomayko/bin/pbindent

## Creating a Patch

### Branches

* `1-x-stable` is the branch that 1.x versions are cut from
* `master` is what will become 2.x

Please consider which branch (or both) should be the target of your feature / bug fix. If both consider having one branch for the patch against `1-x-stable` and one branch for the patch against `master`

### Process

* [Fork](http://help.github.com/forking/) defunkt/resque
* Create a topic branch: `git checkout -b my_fix`
* Make your changes
* Push your branch: `git push origin my_fix`
* Open an [Issue](http://github.com/defunkt/resque/issues) referencing your branch.
* Please do *not* push to `master` on your fork. This will make everyone's life easier. 