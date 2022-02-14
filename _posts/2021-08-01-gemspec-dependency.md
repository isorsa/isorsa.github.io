---
title: Gemspec dependecies with a specific revision/git/submodules etc.
published_at: 2021-08-01 12:13:14
updated_at: 2021-08-01 12:13:14
---

How to specify a specific revision or git path of a dependency in the gemspec? Let's say you are building a gem that has a dependency and your gem relies on another gem with a specific revision/git path or you could be just debugging two gems. But in the gemspec you can only add a version, something like this:

<pre>s.add_dependency "isorsa-utils", "~> 3.0"</pre>

There are situations, when your dependency gem is no longer maintained. You might want to fork the branch and add a patch/fix and use that fork in your gem. Then you can add it to your Gemfile :)

<pre>
source 'https://rubygems.org'

gem 'isorsa', '~> 1.2.3', git: 'https://github.com/isorsa/isorsa-utils.git', submodules: true

# Specify your gem's dependencies in scanning.gemspec
gemspec
</pre>
