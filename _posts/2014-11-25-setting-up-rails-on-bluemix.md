---
layout: post
title: "Setting up Rails on Bluemix"
description: "Set up Rails app on Bluemix hosting platform"
category: 
tags: ["bluemix", "tutorial"]
---
{% include JB/setup %}

# Getting set up on Bluemix, for Rails developers
------------------------------

I will go step by step, please leave a comment below if anything was missing or if the instructions were incorrect. I'm happy to help.


#Bluemix Instructions

* Install [cloudfoundry cli](https://github.com/cloudfoundry/cli)
* Once finished, type `cf -v` in your terminal, and you should see something like this: "cf version 6.6.2-0c953cf-2014-10-17T18:10:36+00:00"
* Note: if you don't see anything, you might need to open a new terminal tab
* Type into your terminal: `rails new MY_APP_NAME -d=postgresql`
* `cd` into your new app folder
* Add `rails_12factor` gem to gemfile for logging purposes
* Run `bundle` in your terminal to install the gem
* Create a new file `manifest.yml` in your project root directory, this will store your Bluemix App config
* Log in to [Bluemix](http://bluemix.net)
* Click **Create an app** <br/>
![create an app](http://i.imgur.com/ZgXYC7T.png)<br/>
* Choose **Ruby on Rails** runtime towards the bottom<br/>
![choose ruby on rails](http://i.imgur.com/2u3ZudM.png)<br/>
* Choose your app name and route and then click **create**
* Click **add a service**<br/>
![add a service](http://i.imgur.com/2SgTk3b.png)<br/>
* Choose **ElephantSQL**<br/>
![elephantsql](http://i.imgur.com/fiYLPZr.png)<br/>
* Rename the service name to `elephant` and click **create**<br/>
![rename service](http://i.imgur.com/5uFYz9a.png)<br/>
* Update your **manifest.yml** to look similar to this:

{% highlight ruby linenos %}
---
applications:
- name: APP_NAME
  memory: 512M
  instances: 1
  host: HOST_NAME
  domain: mybluemix.net
  path: .
  services:
  - elephant
{% endhighlight %}
* In your console run `rake db:create db:migrate` to create your dbs and the schema
* Open **config/database.yml** and change your Production settings to look similar to this:

{% highlight ruby linenos %}
production:
<% if ENV['VCAP_SERVICES'] %>
<% services = JSON.parse(ENV['VCAP_SERVICES'])
   postgres = services["elephantsql"]
   credentials = postgres.first["credentials"] %>
  url: <%= credentials["uri"] %>
<% else %>
  <<: *default
  database: db/production.sqlite3
<% end %>
{% endhighlight %}
* You should now be ready to push to Bluemix
* Run `cf login`
* The API endpoint it's asking for is: **https://api.ng.bluemix.net**
* Choose your target space and org
* Run `cf push -c "rake db:migrate"` (**NOTE: This may take a moment, and also cause an error at the end**)
* If an error was caused from the previous command, hit `ctrl+c` to cancel it
* Next, run `cf push -c "null"` and you shouldn't get any errors
