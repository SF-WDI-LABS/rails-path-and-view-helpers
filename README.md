![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png)
# Rails Path & View Helpers

### Why is this important?
<!-- framing the "why" in big-picture/real world examples -->
*This workshop is important because:*

Rails provides a ton of helper methods to make it easier to write code, but it can take a long time to learn them all. In this workshop, we'll look at a few important helpers that every Rails app should take advantage of.

### Objectives
*After this lesson, developers will be able to:*

- Recognize Rails path helpers and URL helpers.
- Explain benefits of using Rails form helpers and link helpers, and determine correct syntax for them.

### Where should we be now?
*Before this workshop, developers should already be able to:*

- Start a Rails app with a route and controller for a home page view.
- Incorporate routes, a controller, and a model for a single resource.
- Read and write routes.


## Path and URL Helpers

Rails adds helper methods that return the paths for routes in your app.  These are configured through the `config/routes.rb` file.

Here's some `rails routes` output:

```
$ rails routes
     Prefix Verb   URI Pattern                 Controller#Action
    turkeys GET    /turkeys(.:format)          turkeys#index
            POST   /turkeys(.:format)          turkeys#create
 new_turkey GET    /turkeys/new(.:format)      turkeys#new
edit_turkey GET    /turkeys/:id/edit(.:format) turkeys#edit
     turkey GET    /turkeys/:id(.:format)      turkeys#show
            PATCH  /turkeys/:id(.:format)      turkeys#update
            PUT    /turkeys/:id(.:format)      turkeys#update
            DELETE /turkeys/:id(.:format)      turkeys#destroy
```

The **Prefix** column on the left helps us see the available URL and path helpers.  

Based on the `turkeys` prefix from the first row, we can tell that a `turkeys_path` helper exists. This method will return the path that corresponds to the index route for all turkeys (`'/turkeys`).  

Looking further down the line, we can see following helpers exist: `new_turkey_path`, `edit_turkey_path`, `turkey_path`.

In the URI Pattern column, we can see that some of those paths require an **id**.  For those routes, we'll pass an argument to the helper so it can fill in the `:id` portion of the path.  

```
# examples
turkey_path(12)        returns  "turkeys/12/edit"
turkey_path(@turkey)   returns  "turkeys/#{@turkey.id}"
```

You can set individual route prefixes by using `as:` in your routes.

  ```rb
  # inside config/routes.rb
  get 'posts/new', to: 'posts#new', as: 'new_post'
  ```

The turkeys output from above used `resources` in the route configuration, so the Rails auto-generated prefixes for many of the routes. It's good practice to stick to these prefixes when possible even if you're writing out routes by hand instead of using `resources`.

  ```rb
  # inside config/routes.rb
  resources :turkeys
  ```

There are also URL helpers that use the same prefixes to generate a full URL instead of just the path part. For instance, `turkeys_url`. See the Rails Routing Guide [Path and URL helpers section](http://guides.rubyonrails.org/routing.html#path-and-url-helpers).


### Where should you use path helpers?

**Everywhere you want a link or a redirect in your app!**

Here's part of a `posts#create` action:

```ruby
# inside app/controllers/PostsController.rb
def create
  # create a post with params from client and save to post variable
  # THEN ...
  redirect_to "/posts/#{post.id}"
end
```

The method will create a post then redirect to that post's show page (or try to - there's no error handling!).

Instead of writing out `redirect_to "/posts/#{post.id}"`, we should take advantage of the `post_path` helper:

```ruby
  redirect_to post_path(post.id)
```

Rails can pull out the id for us.

```ruby
  redirect_to post_path(post)
```

There's also an even shorter syntax!
```ruby
  redirect_to post
```



## View Helpers

Rails provides a huge swath of helpers designed to make it more convenient to generate HTML for your views, especially HTML related to your resources.  These view helpers also enforce the Rails way by automatically setting some attributes inside the HTML.  

Let's start by looking at some simple and very commonly used view helpers.

#### `link_to` and `button_to`

Pair up. One person should silently skim the documentation for [`link_to`](http://api.rubyonrails.org/classes/ActionView/Helpers/UrlHelper.html#method-i-link_to).   The other should silently skim the documentation for
[`button_to`](http://api.rubyonrails.org/classes/ActionView/Helpers/UrlHelper.html#method-i-button_to).

After 2 minutes, turn to your partner and explain what kind of element your method generates in the HTML and what kind of request we can expect that element to send.

As a team, list differences between `link_to` and `button_to`.  Come up with an example of where we could use each in a blog app.


#### `link_to` examples

Simple link:
```rb
link_to "Profile", @profile
# => <a href="/profiles/1">Profile</a>
```

Setting class and id in HTML:

```ruby
link_to "Articles", articles_path, id: "news", class: "article"
# => <a href="/articles" class="article" id="news">Articles</a>
```

#### `button_to` example

Image delete button

```erb
<%= button_to "Delete Image", { action: "delete", id: @image.id },
                                method: :delete, data: { confirm: "Are you sure?" } %>

# => "<form method="post" action="/images/delete/1" class="button_to">
#      <input type="hidden" name="_method" value="delete" />
#      <input data-confirm='Are you sure?' value="Delete Image" type="submit" />
#      <input name="authenticity_token" type="hidden" value="10f2163b45388899ad4d5ae948988266befcb6c3d1b2451cf657a0c293d605a6"/>
#    </form>"
```

> Remember: run `rails routes` and look at the Prefix column to see what `_path` helpers are available.

| Path helper          | using this path helper with link_to in erb |
|---------------------| -------------------------------------------|
| `turkeys_path`        | `link_to "view all turkeys", turkeys_path` |
| `new_turkey_path`     | `link_to "create a new turkey", new_turkey_path` |
| `edit_turkey_path`    | `link_to "edit this turkey", edit_turkey @turkey` |
| `turkey_path`         | `link_to "view this turkey", turkey_path @turkey` |

##### PUT, PATCH, DELETE

Some browsers don't support PUT, PATCH & DELETE as form submission methods.  Rails however has a _work-around_ for this.  

Rails adds a hidden input field with the name `_method`. Rails sets the actual method of the form to "post" but internally changes the request type based on the hidden field before the request gets to the routes.  

### Form Helpers

As we saw with `button_to`, Rails can generate forms for us.  There are two main kinds of helpers we can use to generate forms:  [FormTagHelper](http://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html) and [FormBuilderHelper](http://api.rubyonrails.org/classes/ActionView/Helpers/FormBuilder.html).  You can read more about both in the [Form Helpers Rails Guide](http://guides.rubyonrails.org/form_helpers.html)

* FormTagHelper's `form_tag` is very general, and it's often used for method/action combinations that don't map directly to a RESTful route for one of our resources.

```erb
<%= form_tag("/search", method: "get") do %>
  <%= label_tag(:q, "Search for:") %>
  <%= text_field_tag(:q) %>
  <%= submit_tag("Search") %>
<% end %>
```

Generated html:

```html
<form accept-charset="UTF-8" action="/search" method="get">
  <input name="utf8" type="hidden" value="&#x2713;" />
  <label for="q">Search for:</label>
  <input id="q" name="q" type="text" />
  <input name="commit" type="submit" value="Search" />
</form>
```

* FormBuilderHelper's `form_for` is intended to work with our resources and RESTful routes. It does a lot to help you pre-fill values.

```erb
<%= form_for @article, url: {action: "create"}, html: {class: "nifty_form"} do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :body, size: "60x12" %>
  <%= f.submit "Create" %>
<% end %>
```
Generated html:

```html
<form accept-charset="UTF-8" action="/articles" method="post" class="nifty_form">
  <input id="article_title" name="article[title]" type="text" />
  <textarea id="article_body" name="article[body]" cols="60" rows="12"></textarea>
  <input name="commit" type="submit" value="Create" />
</form>
```

#### Form Builder and `form_for`

We'll mostly be working with resources, so it's important to get some exposure to `form_for`.

1) You build up a form in ERB.

```erb
<%= form_for @article do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :body, size: "60x12" %>
  <%= f.submit "Submit" %>
<% end %>
```

Inside a form builder, you'll build up a form using an object.  That's the parameter in the  `form_for` `do`... `end` block. It's usually called `f`.

Each line inside the form will generate a label or an input.  Most of these `f.____` methods take in a symbol. That symbol tells the form builder which attribute of the model is being input in that field.  Looking at the form builder above, we can tell that the `@article` variable has a `title` attribute and a `body` attribute.

2) Rails uses this code to generate a form for you!

Note that when the HTML is generated, the **name** HTML attribute for each form `input` will be a combination of the model name and the attribute symbol.

```html
<form accept-charset="UTF-8" action="/articles" method="post">
  <input id="article_title" name="article[title]" type="text" />
  <textarea id="article_body" name="article[body]" cols="60" rows="12"></textarea>
  <input name="commit" type="submit" value="Create" />
</form>
```

The name HTML attribute is important to us in forms because these names become the keys in the data the client sends to the server.   

3) You get the data in your controller!

In Rails, we access this data in a controller through the `params` hash.  In the example above, the controller will receive a `params` hash that looks like this:

```rb
{
  'article' => {
                 'title' => '#whatever value the title text input had when submitted',
                 'body' => '#whatever value the body textarea had when submitted'
               }
}
```

> It's a best practice to use "strong parameters," which is a pattern of using built-in methods to say what format of parameters your app will accept. With the example above, we'd want to write the following in our controller code:

```rb
private

# Using a private method to encapsulate the permissible parameters
# is a good pattern since you'll be able to reuse the same
# permit list between create and update.
def article_params
  params.require(:article).permit(:title, :body)
end
```

Rails 5.1  also added [`form_with`](http://edgeguides.rubyonrails.org/5_1_release_notes.html#unification-of-form-for-and-form-tag-into-form-with), which can be used in resource-based situations like `form_for` and for more general forms like `form_tag`.

#### Independent Practice: View Helper Research

Research one of the methods below, and prepare to give a 2 sentence explanation about where it is used and what it does to everyone!


1. `csrf_meta_tags`    
2. `stylesheet_link_tag`  
3. `javascript_include_tag`  
4. `audio_tag`
5. `image_tag`
6. `image_url`
7. `video_tag`
8. `time_ago_in_words`
9. `hidden_field` or `f.hidden_field`
10. `password_field`
11. `color_field`
12. `f.label`
13. `check_box`
14. `collection_check_boxes`
15. `select` (form builder)
16. `date_field`
17. `date_select`
18. `truncate`
19. `pluralize`
20. `simple_format`
21. `number_to_human`

### Closing Thoughts

- How can we figure out what path helpers are available?
- Why would we use `form_for` and `link_to`?
- Where would you look for syntax for Rails form helpers?


### Additional Resources

* [Glossary](glossary.md)  
* Rails Guides:
  * [Action View Overview](http://guides.rubyonrails.org/action_view_overview.html)
  * [Form Helpers](http://guides.rubyonrails.org/form_helpers.html)
* API documentation:
  * ActionView's [FormHelper](http://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html)
  * ActionController's [`redirect_to`](http://api.rubyonrails.org/classes/ActionController/Base.html#class-ActionController::Base-label-Redirects)
