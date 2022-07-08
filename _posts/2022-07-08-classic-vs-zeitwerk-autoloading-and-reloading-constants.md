---
title: Classic vs Zeitwerk autoloading
published_at: 2022-07-08 12:13:14
updated_at: 2022-07-08 12:13:14
---

<h4>WHAT?</h4>
<p>
In Rails 6 and later versions the autoloading was replaced by a Zeitwerk gem. You don't really need to know about Zeitwerk gem, unless you use it as a dependency in your gems or applications or you upgrade your application from Rails 5 to Rails 6. Zeitwerk is a gem that does not have any dependencies.
</p>
<p>Zeitwerk provides 3 features: <code>autoloading, eager loading and reloading</code></p>
<pre>
# config/application.rb
config.load_defaults "6.0" # enables Zeitwerk
</pre>

<h4>WHY?</h4>
<p>
  <ol>
    <li>Improve Rails autoloading (performance)</li>
    <li>DRYness (file names match constant paths)</li>
    <li>Writing requires manually is brittle (ruby apps, gems)</li>
  </ol>
</p>

<br />
<h4>Differences with Classic Mode</h4>

<img class="img-responsive img-thumbnail" width="80%" src="img/services/2022-07-08/zeitwerk_vs_classic.png" alt="zeitwerk" />

<br />
<p>In Rails application we don't have to worry about requiring or loading its classes (as opposite to Ruby apps, gems or React apps). For example, when we add a new controller PostsController, we don't have to require ApplicationController or Post model. Everything is autoloaded for us, unless we need to load stuff from lib/ directory.</p>

<pre>
  class PostsController < ApplicationController
    def index
      @posts = Post.all
    end
  end
</pre>

<p>In PostsController, constants defined with <code>class</code> and <code>module</code> keywords will not trigger autoloading, but ApplicationController is unknown, the constant is missing and Rails will try to autoload the constant. Rails iterates over autoload_paths.</p>

<p>Rails looks up constants in the collection called autoload_paths and by default it contains all subdirectories of app/, eager_load_paths is initially the app/ paths too.</p>

<p>The value of autoload_paths can be inspected by running: </p>
<pre>rails runner 'puts ActiveSupport::Dependencies.autoload_paths'</pre>

<br />
<h4>Using Zeitwerk in a gem</h4>

<pre>
# In terminal

| => bundle gem isorsa
________________________________________________________________________________

# isorsa.gemspec

Gem::Specification.new do |spec|
  spec.name          = "isorsa"
  spec.version       = Isorsa::VERSION
  spec.authors       = ["Sergey Kim"]
  spec.email         = ["kim@isorsa.com"]

  spec.add_dependency "zeitwerk", "~> 2.6"
end

________________________________________________________________________________

# isorsa.rb

<strike>require_relative "isorsa/version"</strike>

require "zeitwerk"
loader = Zeitwerk::Loader.for_gem
loader.setup # ready!

module Isorsa
  class Error < StandardError; end
end

________________________________________________________________________________

# In terminal

| => gem build isorsa.gemspec
| => gem install ./isorsa-0.1.0.gem
| => irb
irb(main):001:0> require 'isorsa'
=> true
irb(main):002:0> Isorsa::VERSION
=> "0.1.0"
</pre>

<br />
<h4>How autoloading works?</h4>

<p>If there is nesting, which will be the case with namespaces. For example:</p>

<pre>
module Admin
  class UsersController < ApplicationController
    def index
      @admin_users = User.admin.all
    end
  end
end
</pre>

<p>In that case, to autoload User constant Rails will try to autoload:</p>
<pre>
Admin::UsersController::User
Admin::User
User
</pre>

<p>Rails looks in autoload_paths the following file names:</p>
<pre>
admin/users_controller/user.rb
admin/user.rb
user.rb
</pre>

<p>When the lookup fails, Rails starts the lookup in the parent namespaces.</p>
