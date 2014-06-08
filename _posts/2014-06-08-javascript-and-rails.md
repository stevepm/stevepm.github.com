---
layout: post
title: "Javascript and Rails"
description: "Using the built in Javascript in Rails"
category: 
tags: ["Javascript", "Rails", "JQuery"]
---
{% include JB/setup %}

## Working with Javascript and rails

It turns out [Rails](http://rubyonrails.org/) comes standard with some excellent and well [documented](http://guides.rubyonrails.org/) functionality. Who knew? In an effort to make my newest project, [GemBundle](http://www.gembundle.com), a bit more robust, I've decided to use some basic Javascript, with the help of some built in Rails methods.

### The problem I was trying to solve
* When a user clicked a heart (to indicate they like a gem), the page would reload, after performing some functions and changing the css.
    * This is a problem, because they might be halfway down the page and still browsing, and now the user has to find where they were again. Also, items may have moved around, due to the fact that items are sorted based on the number of likes they have.
* Make my website more user friendly.
* Learn how to use Javascript with Rails.

---------

## Step 1

Let's take a look at the current code and see how we integrate Javascript.

#### The View
`_heart_button.html.erb`
{% highlight erb linenos %}
<% if logged_in? %>
  <% if !(current_user.voted_up_on? gem) %>
    <% path = like_gem_path(gem.id, current_user.id) %>
    <% button = 'btn-default' %>
  <% else %>
    <% path = unlike_gem_path(gem.id, current_user.id) %>
    <% button = 'btn-warning' %>
  <% end %>
  <%= link_to path, method: :post, class: "btn #{button} btn-sm active", id: "heart_#{gem.name}" do %>
    <span class="glyphicon glyphicon-heart"></span>
    <input type="hidden" value="<%= gem.id %>" name="popular_gem_id">
    <input type="hidden" value="<%= current_user.id %>" name="user_id">
  <% end %>
<% end %>
{% endhighlight %}

#### The Controller
`popular_gems_controller.rb`
{% highlight ruby linenos %}
def like
    user = User.find(params[:user_id])
    gem = PopularGem.friendly.find(params[:id])
    user.likes gem
    redirect_to :back
  end

def unlike
    user = User.find(params[:user_id])
    gem = PopularGem.friendly.find(params[:id])
    user.dislikes gem
    redirect_to :back
end
{% endhighlight %}

---------
1. First we are checking to see if a user is logged in (otherwise they aren't allowed to vote).
2. Next we check to see if the current user that's logged in has already voted for (or liked) the gem.
    
    * If they haven't:
      1. We set the color of the heart to the default grey.
      2. We also will send them to the "like" action in the popular_gems_controller if they click it.
    * If they have:
      1. We set the color of the heart to red.
      2. And we send them to the "unlike" action.
      
---------

## Step 2

In order to figure out how to use Javascript in Rails, I went to the guides and found [this](http://guides.rubyonrails.org/working_with_javascript_in_rails.html#server-side-concerns). 

So let's make some edits and then I'll explain it after:

----------

#### The View
`_heart_button.html.erb`
{% highlight javascript linenos %}
<script>
  $(function() {
    return $("a[data-remote]").on("ajax:success", function(e, data, status, xhr) {
      var id, likes;
      id = "heart_" + data.gem.name;
      likes = data.likes;
      if (likes) {
        $("#" + id).removeClass("btn-default");
        $("#" + id).addClass("btn-warning");
      } else {
        $("#" + id).removeClass("btn-warning");
        $("#" + id).addClass("btn-default");
      }
    });
  });
</script>

<% if logged_in? %>
  <% if !(current_user.voted_up_on? gem) %>
    <% button = 'btn-default' %>
  <% else %>
    <% button = 'btn-warning' %>
  <% end %>
  <%= link_to like_gem_path(gem.id, current_user.id), method: :put, class: "btn #{button} btn-sm active", id: "heart_#{gem.name}", remote: true do %>
    <span class="glyphicon glyphicon-heart"></span>
    <input type="hidden" value="<%= gem.id %>" name="popular_gem_id">
    <input type="hidden" value="<%= current_user.id %>" name="user_id">
  <% end %>
<% end %>
{% endhighlight %}

#### The Controller
`popular_gems_controller.rb`
{% highlight ruby linenos %}
def like
  gem = PopularGem.friendly.find(params[:id])
  if current_user.liked?(gem)
    current_user.dislikes gem
    current_user.subtract_points(1)
  else
    current_user.likes gem
    current_user.add_points(1)
  end
  render json: {gem: gem, likes: current_user.liked?(gem)}
end
{% endhighlight %}

-----------

## The solution

#### The Controller
1. Combined the two actions, like and unlike, into a single action, like.
2. Now when the heart gets clicked, the like action is called and we do a simple check to see if the user already liked the gem and update it accordingly.
3. At the end, you see we render JSON. This is very important.
4. Passing into the JSON are two things:
 * The current gem we are looking at.
 * A boolean that states whether or not the user likes the gem.

#### The View
This is a bit more complicated
1. On line 24, we added the option, `remote: true`. This adds a `data-remote="true"` to the html anchor tag, which tells the information to be sent through Ajax rather than the normal link mechanism.
2. Now, when someone clicks on the heart, the like action in the controller is called and then, as mentioned previously, the controller renders the json, sending that information back to the page.
3. The information we receive gets sent through the `<script>` section.
4. First we are assigning empty variables, `id` and `likes`.
5. Next we set these variables.
6. We set `id` equal to what the id attribute of the html link is, in order for us to update it.
7. We set `likes` equal to the boolean which represents whether the user likes the gem.
8. Next we just check if they like it, and if they do, we remove the css class that set the color to grey and add the css class that sets the color to red.
9. Otherwise, we just do the opposite.


### There you have it

As you can see, it's fairly simple to manipulate the DOM. 
Once you are able to get the data you need, all you have to do is write a simple conditional that will update your CSS.