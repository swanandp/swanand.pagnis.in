---
layout: post
title: "Console your gems: Add REPL to them"
date: 2014-04-28 12:27:34 +0530
comments: true
categories: rake repl
---

When you are working on a Ruby gem or a Ruby library ( why you would have a Ruby library that isn't a gem is beyond me ), it is <s>always</s> often desirable to have a Pry session loaded with the gem your working on.  In fact, I'll go out on a limb and say that REPL driven development is a a must-do when writing libraries.  REPLs are like ice added to your beer when it isn't cold anymore, except this ice is made from the same beer.

A lot of gems ship with this functionality, but if not, it is extremely easy to add one.  To avoid polluting every gem with our code, we can utilize the concept of Rake's global tasks.  Rake's default source for looking at tasks or <span title="Google it">rules</span> is the Rakefile and all files that are declared as source files in the Rakefile; typically `*.rake` files in your tasks folder, but this can vary depending on your project.  However, rake also looks at `~/.rake/*.rake` if we ask it to. So let's create a file called `~/.rake/console.rake` and add the following task to it:


```ruby
    desc "Open a pry (or irb) session preloaded with this gem"
    task :console do
      begin
        require 'pry'
        gem_name = File.basename(Dir.pwd)
        sh %{pry -I lib -r #{gem_name}.rb}
      rescue LoadError => _
        sh %{irb -rubygems -I lib -r #{gem_name}.rb}
      end
    end
```

And run our shiny new rake task:

```
    ➜  awesome_sauce git:(master) ✗ rake console
    rake aborted!
    Don't know how to build task 'console'

    (See full trace by running task with --trace)
```

Bummer! Let's double check:

```
    ➜  awesome_sauce git:(master) ✗ rake -T console


```

Nothing! What gives?  Not a cause of worry, because this is the expected behaviour, and we have a failing test. Rake takes global pollution quite seriously, and *does not* load the global tasks unless asked to.  So let's ask rake to do so, by adding a `-g` flag.  Because, you know, g for global:

```
    ➜  awesome_sauce git:(master) ✗ rake -gT console
    rake console  # Open a pry (or irb) session preloaded with this gem

```

And subsequently

```
    ➜  awesome_sauce git:(master) ✗ rake -g console
    pry -I lib -r awesome_sauce.rb
    2.1.1 (main):0 >
    # Just to be sure
    2.1.1 (main):0 > ^D
    ➜  awesome_sauce git:(master) ✗ cd ../secret_sauce
    ➜  secret_sauce git:(hush-hush) ✗ rake -g console
    rbx-2.1.1 (main):0 >

```

With this, I bow out and promise to come back with more palatable content later.
