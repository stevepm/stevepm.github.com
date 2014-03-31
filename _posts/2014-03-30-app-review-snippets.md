---
layout: post
title: "App Review: Snippets"
description: "Reviewing snippets app"
category:
tags: [App Review]
---
{% include JB/setup %}

## Snippets
---

[Snippets](http://snippets.me/) is an awesome tool that I've been using recently
to help me keep track of what I'm learning.

Snippets provides an easy to use interface for storing and reviewing bits and
pieces of code that you find useful. The GUI is very simple and easy to learn.
![Snippets interface]({{ site.url }}/assets/images/snippets_interface.png "Snippets GUI")

Most recently, I've been using snippets to store the necessary code needed to
start writing a [Sinatra](http://www.sinatrarb.com/) application. As you can see below,
I have created a group called **Sinatra**. Within the group, I have several files,
each containing the necessary snippets in order to
start deploying a functional Sinatra web application, fully equipped with
[RSpec](http://rspec.info/) and [Capybara](https://github.com/jnicklas/capybara)
test functionality.
![Snippets groups]({{ site.url }}/assets/images/snippets_groups.png "Snippets Groups")

I'm very excited to see what the future holds for Snippets, as it has become an invaluable tool for myself.
A few things that I would love to see in future releases are:
1. Some sort of view to see popular snippets per language - And the ability to favorite/add those snippets to your account.
2. Exporting/importing groups of snippets - for sharing purposes.
3. Improved functionality on the *Snippets Assistant*.

Go ahead and [download](http://snippets.me/download) it to see if you like it. It's currently available for both Mac and Windows.
![Snippets interface]({{ site.url }}/assets/images/snippets_downloads.png "Download Snippets")

Also, go ahead and follow them on twitter at [@snippetsme](https://twitter.com/snippetsme)

If you're interested in my Sinatra snippets, here they are:

### Config.ru
---
{% highlight ruby linenos %}
require './app'

run App
{% endhighlight %}
### app.rb
---
{% highlight ruby linenos %}
require 'sinatra/base'

class App < Sinatra::Application
{% endhighlight %}
### Spec_Helper
---
{% highlight ruby linenos %}
ENV['RACK_ENV'] = 'test'
{% endhighlight %}
### Within your spec file
---
{% highlight ruby linenos %}
require 'spec_helper'
require 'capybara/rspec'
require_relative '../myapp'
Capybara.app = App
{% endhighlight %}
### Gems in Gemfile
---
{% highlight ruby linenos %}
group :development do
    gem 'rspec', '~> 2.14.1'
    gem 'capybara', '~> 2.2.1'
    gem 'launchy', '~> 2.4.2'
end
gem 'sinatra', '~> 1.4.4'
{% endhighlight %}
