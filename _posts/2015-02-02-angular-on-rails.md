---
layout: post
title: "Angular on Rails part 1"
description: "Setting up Angular on top of Rails"
category: 
tags: ["Angular", "Rails", "Testing"]
---
{% include JB/setup %}

# Angular on Rails

Hello there! Today we're going to set up a Rails application that uses Angular to serve the front end. Additionally, we'll walk through testing of both the back-end and the front-end.

## Technology

### We will use a number of technologies to accomplish our goal
---

**Rails (ruby)** - The back-end

**RSpec (ruby)** - The back-end testing framework

**capybara & selenium (ruby)** - The end to end browser tests

**Jasmine (javascript)** - The front-end testing framework

**Bower** - The front end package manager (similar to bundler)

**Twitter Bootstrap** - The css styling framework

**Heroku** - The PaaS for deployment

--

## What we are building

We will build a very simple CRUD application that will demonstrate:

1. How to render JSON in the Rails Controller
2. How to test Rails Controllers
3. How to use Bower to pull in front-end dependencies (such as Angular and Bootrap)
4. How to add those front-end dependencies to the Rails Asset pipeline
5. Testing the whole application from front to back
6. POSTing, PUTing, GETing and DELETEing in an Angular app

## Let's get started!

**create the application using postgresql as our database and removing minitest in favor of RSpec**<br/>
$ `rails new learn-angular -d=postgresql -T`<br/>
$ `cd learn-angular`<br/>

--
### Add testing dependencies
Open up `Gemfile` and do the following:<br/>
**Remove:**<br/>

{% highlight ruby  %}
gem 'turbolinks'
{% endhighlight %}

**Add:**<br/>

{% highlight ruby  %}
group :test, :development do
	gem 'rspec-rails'
	gem 'capybara'
	gem 'database_cleaner'
	gem 'selenium-webdriver'
end
{% endhighlight %}

And then:<br/>
$ `bundle`

### Create your DBs
$ `rake db:create db:migrate`<br/>

--
### Setting up Bower for front-end package management
We're going to use a gem *bower-rails* to set up bower in our application. It provides a very nice way to manage dependencies that is very similar to a `Gemfile`.

Go back to your `Gemfile` and **add:**<br/>

{% highlight ruby  %}
gem 'bower-rails'
{% endhighlight %}
$ `bundle`

Now we need to install Bower so that we can add Angular and Bootstrap<br/>
$ `touch Bowerfile` <br/>
Open `Bowerfile` and **add:**<br/>

{% highlight ruby  %}
asset 'angular'
asset 'bootstrap-sass-official'
{% endhighlight %}

$ `rake bower:install` <br/>

We now need to make sure that these new files get picked up by the **Rails Asset Pipeline** <br/>

To do so, open up `config/application.rb`<br/>
We're going to add the bower folders to the assets path. We're also going to make sure that the fonts from Bootstrap get precompiled.

Inside of:

{% highlight ruby  %}
class Application < Rails::Application


end
{% endhighlight %}

add

{% highlight ruby  %}
config.assets.paths << Rails.root.join("vendor", "assets", "bower_components")
config.assets.paths << Rails.root.join("vendor", "assets", "bower_components", "bootstrap-sass-official", "assets", "fonts")

config.assets.precompile << %r(.*.(?:eot|svg|ttf|woff|woff2)$)
{% endhighlight %}

We now need to load these dependencies in our `app/assets/javascripts/application.js` and `app/assets/css/application.css.scss` files.

Open up `application.js`.

**remove:**<br/>

{% highlight javascript %}
//= require turbolinks
{% endhighlight %}

**add:**<br/>

{% highlight javascript %}
//= require angular/angular
{% endhighlight %}

Open `application.css.scss` and **add:**

{% highlight css %}
@import "bootstrap-sass-official/assets/stylesheets/bootstrap-sprockets";
@import "bootstrap-sass-official/assets/stylesheets/bootstrap";
{% endhighlight %}

--
### Making sure all of the pieces are working

Let's make sure the app is working and configured properly.

In `config/routes.rb`, **add**:

{% highlight ruby  %}
root 'home#index'
{% endhighlight %}

Next, create the `HomeController` in `app/controllers/home_controller.rb` and add the following code to it:

{% highlight ruby  %}
class HomeController < ApplicationController
  def index
  end
end
{% endhighlight %}

Next, we're going to create our Angular app. Create `app/assets/javascripts/angular/app.js` and add the following code to it:

{% highlight javascript  %}
app = angular.module('app', []);
{% endhighlight %}

Create the view, in `app/views/home/index.html` and place in the following:

{% highlight html  %}
<h1 ng-if="name">Hello, {{name}}</h1>
<form>
	<input type="text" ng-class="name">
</form>
{% endhighlight %}

Run the server:<br/>
$ `rails s`

Visit `localhost:3000` and voila! Angular two-way binding should be working now.

--

That's it for part 1. Next, we'll work on test driving the application to CRUD some objects.