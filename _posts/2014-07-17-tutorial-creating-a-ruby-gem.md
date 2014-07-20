---
layout: post
title: "Tutorial: Creating a Ruby Gem"
description: "Creating a Ruby Gem using TDD and RSpec"
category: 
tags: ["tutorial", "gem", "rspec"]
---
{% include JB/setup %}

# Building your first Ruby Gem
------------------------------

    *PreReqs*

* Have bundler installed
* Understand what RSpec is and why it's important
* Have basic knowledge of Ruby

-------------------------------

I say 'first' because this is geared towards people who probably have never built a gem before, and thus it will be pretty basic.
This gem will go to a webpage and return you the title of the webpage.
The first thing you want to do is create your new gem.

* `bundle gem my_gem`
* `cd my_gem`
* Open folder with your text editor

We are going to test drive this gem all the way through.

Let's set up our gemspec now. The gemspec is for gem specific information.

* `spec.name` -- The name of your gem
* `spec.version` -- This links to a module which specifies your gem version. (**NOTE: Needs to be updated every time you release new content**)
* `spec.authors` -- Authors of the gem, in an array.
* `spec.email` -- Email address(es) of the author(s)
* `spec.summary` -- Short summary of the what the gem does
* `spec.description` -- Long summary of the what the gem does
* `spec.homepage` -- I usually put the online repository where the gem is stored

Now we want to set up the gems our gem will be using.

Add the following at the bottom of your gemspec:

{% highlight ruby linenos %}
  spec.add_development_dependency "jazz_hands"
  spec.add_development_dependency "rspec"
  spec.add_development_dependency "vcr"
  spec.add_development_dependency "webmock"
  spec.add_dependency "nokogiri"
{% endhighlight %}

* `add_development_dependency` - Is used for gems that you need during development 
* `add_dependency` - Is used for gems that you need during development/production

Now, run `bundle` in your console
Next, run `rspec --init` to create your spec_helper and your spec folder.

Open `.rspec` and delete '--warnings'.

Create a file `spec/my_gem_spec.rb` and place the following inside:

{% highlight ruby linenos %}
require 'spec_helper'

describe MyGem do
  it 'queries a website and returns the title' do
    google = MyGem.new("http://www.google.com")
    expect(google.title).to eq("Google")
  end
end
{% endhighlight %}

Open `my_gem.gemspec` and change line 8 to be:
{% highlight ruby linenos %}
  spec.version       = MyGemVersion::VERSION
{% endhighlight %}

Open lib/my_gem/version.rb and change line 1 to be:
{% highlight ruby linenos %}
module MyGemVersion
{% endhighlight %}

Open lib/my_gem.rb and paste the following in:

require "my_gem/version"
{% highlight ruby linenos %}
class MyGem
  def initialize(url)
    @url = url
  end
end
{% endhighlight %}

Now, run `rspec` in terminal and you should get the following error:

`undefined method 'title' for #<MyGem:0x007f83826c5ab8 @url="http://www.google.com">`

Great, so now we need to actually create the method that queries the page and returns the title:

Update lib/my_gem.rb to be:

{% highlight ruby linenos %}
require "my_gem/version"
require 'nokogiri'
require 'open-uri'

class MyGem
  def initialize(url)
    @url = Nokogiri::HTML(open(url))
  end

  def title
    @url.title
  end
end
{% endhighlight %}

And run `rspec` again and your tests should be passing.

Now, let's add VCR so that we don't have to hit google.com everytime we run our specs.

Open `spec/spec_helper.rb` and paste in the following:

{% highlight ruby linenos %}
require 'my_gem'
require 'vcr'
require 'webmock'

RSpec.configure do |config|
  VCR.configure do |c|
    c.cassette_library_dir = 'spec/fixtures/vcr_cassettes'
    c.hook_into :webmock # or :fakeweb
  end
end
{% endhighlight %}

Open `spec/my_gem_spec.rb` and paste in the following:

{% highlight ruby linenos %}
require 'spec_helper'

describe MyGem do
  it 'queries a website and returns the title' do
    VCR.use_cassette('/lib/my_gem/google') do
      google = MyGem.new("http://www.google.com")
      expect(google.title).to eq("Google")
    end
  end
end
{% endhighlight %}

Run `rspec` again, it should be green.

There are many other fun things you can do with this gem, such as:

* return the body content
* return all of the links on the page
* get the metadata
* basically any information that is on the page

# Releasing the gem

At this point, you may want to release the gem (sending it to RubyGems.org).

* `rake build` - builds the gem
* `rake install` - installs it on your system
* `rake release` - sends it to RubyGems.org for others to use

If you would like to see my final code, you can view it on GitHub here:
[my_gem](https://github.com/stevepm/gem_tutorial)