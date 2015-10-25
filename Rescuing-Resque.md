# Rescuing Resque

## TL;DR
* change master from unreleased 2.0 to `1-x-stable`
* clean up tests - make the development experience enjoyable
* integrate pending Pull Requests

Resque is a great library for background job execution, but the project needs some love.  Have you used Resque in the past for your background job needs?  Then please, chip in some time reviewing PRs or improving test code!

Here is a tentative list of what needs to be done (comments welcome):

## Make `1-x-stable` master again

[Many](https://github.com/resque/resque/issues/1175) [have](https://github.com/resque/resque/issues/976) stated that the README on the master branch has led them astray, because the published gem is still at 1.25.2.

We will plan on making this switch in the next couple of weeks.  If you feel this is the wrong move, please comment [HERE][IDK].

## Fix up the test suite

Tests need to be fast, and clean, so that there is more incentive to contribute.

One possible approach is using one of the available Redis mocking libraries (mock_redis, fakeredis) in non-forking tests, which could speed suite execution up considerably.

## Review Pending `1-x-stable` PRs

This is the lion's share of the work, as there are currently 56 (that's [*fifty six*](https://github.com/resque/resque/pulls)) open pull requests...

*And that's an awesome problem!*  Let's make use of them!  These PRs need to be evaluated and closed, so that the project can benefit from the community. 

## Decide What to do with 2.0

This is a much less well defined objective.  Given the fact that Resque has been in maintenance mode for years, perhaps the focus should be on stability, with an eye towards legacy support.  

It depends on if there is a perception of Resque fitting a need that Sidekiq and others do not.  

Either way, a major version could be considered a means of removing deprecation warnings on already functional code; this approach neatly avoids the ill-fated [Second System Effect](https://en.wikipedia.org/wiki/The_Mythical_Man-Month#The_second-system_effect).  

## Epilogue

Please help us keep a still-viable, process-oriented background library vibrant for your fellow Rubyists!
