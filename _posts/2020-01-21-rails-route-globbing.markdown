---
layout: post
comments: true
title: "Rails Route Globbing"
date: 2020-01-21 00:00:00 -0500
categories: rails
---

Using a wildcard syntax (`*`) when defining routes in `routes.rb`, we can slurp 
up URL segments and make them available in our controller `params`. Here's an 
example: 

### Globbing Example

```ruby
# config/routes.rb

get 'photos/*path', to: 'photos_controller#index'
```

Now in the `PhotosController` you can access the globbed segments in the params 
hash. Let's say the request comes in on `photos/familiy/cousins`.

```ruby
def index
  path = params[:path] # 'family/counsins'
end
```

### Catch-all Route
I can't think of many reasons why we'd want to use route globbing. 
If you know some good used cases, please comment below. A catch-all route is one
possible use case.

Rails will use the first defined route starting from the top of `routes.rb` that 
matches a request URL. So, if a wildcard, globbed route is defined last, this will 
be used as a catch-all route for a request that doesn't match any of the 
previously defined routes.


```ruby
# config/routes.rb at the bottom

match '*path', to: 'static_pages_controller#not_found', via: :all
```

Any request that comes in on a URL that doesn't match any other previously 
defined route, will be routed to the `StaticPagesController`'s `not_found` 
action. The `via: :all` ensures that it will match against any HTTP verb.
Alternatively, we can be explicit about which HTTP verbs we want to match by
using any array of verbs: `via: [:get, :edit]`.

Or if you're too lazy to make a controller and view, you can give it a proc or 
lambda.

```ruby
# config/routes.rb at the bottom

match '*path', to: ->(env) { [404, {}, ['Not Found']] }, via: :all
```

**Side Note**
This works because routes are losely coupled to controllers. Hence, a request
can be handled any object that responds to `.call` and takes a single argument.

### Resources
- [Verb Contraints](https://guides.rubyonrails.org/routing.html#http-verb-constraints)
- [Route Globbing](https://guides.rubyonrails.org/routing.html#route-globbing-and-wildcard-segments)
- If you're into code speluking, and want to deep dive into how route globbing is
implemented, [this method in the Rails router source code](https://github.com/rails/rails/blob/6-0-stable/actionpack/lib/action_dispatch/routing/mapper.rb#L230-L240) 
might be a good place to start.
