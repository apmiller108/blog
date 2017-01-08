---
layout: post
comments: true
title:  "Rails Authenticity Token and Mobile Safari"
date:   2017-01-07 18:26:00 -0500
categories: rails
---

At [Trim Agency][trim agency], we typically architect our web apps by building a Ruby on Rails API separate from an Angular single page font-end app.  Recently, however, we started work on a brown field project that was fully Rails, front-end as well.  After deploying and getting close to one hundred users, I was surprised to see an influx of `ActionController::InvalidAuthenticityToken` exceptions.  I didn’t take long to see the common thread between these exceptions:  'iPhone' or 'Mobile Safari' was consistently appearing in the [User-Agent header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent).

I discovered this problem is fairly common, after I read this [Rails Github issue](https://github.com/rails/rails/issues/21948). Here I will summarize the problem and demonstrate the solution that worked for us.

### What is the  *Authenticity Token*?  

It’s a counter measure to [cross-site request forgery (CSRF)](https://en.wikipedia.org/wiki/Cross-site_request_forgery).  Rails will include the `authenticity_token` in all forms in a hidden input field. For AJAX requests the token is placed in a header called `X-CSRF-Token`.  The `authenticity_token` is verified on the server for all non-GET requests in order to ensure that the request came from the user and not someone else.  How does it do that?

  The `authenticity_token` is also stored in session (`session[:_csrf_token]`) which is encrypted in the client’s cookie.  Since the client’s cookie is automatically sent to the server with a request, Rails can compare the `authenticity_token` in the client’s cookie with the `authenticity_token` submitted in the form. If the tokens match, the request can proceed.  If they don’t match…well, let’s see what can happen.

### Request Forgery Protection
For non-APIs, rails will, by default, add the following to the ApplicationController:
{% highlight ruby %}
class ApplicationController < ActionController::Base
    protect_from_forgery with: :exception
end
{% endhighlight %}
So, when the tokens do not match, Rails will raise the `ActionController::InvalidAuthenticityToken` exception which will stop the request.  How you decide to handle it, is up to you.  As you will see, legitimate users can get this error, so you’ll want to handle it somehow.  For example, you might redirect the user to an error page with a 422 response.

### What's Going On With Mobile Safari?

I was able to reproduce the error easily on mobile Safari on iPhone using the following steps:

1. Log into the app
2. Sign out of the app so I will be redirect back to the log in screen
3. Close the browser (force close it using the home double tap and swipe method)
4. Reopen Safari and try to log in.

After submitting my credentials, the `ActionController::InvalidAuthenticityToken` was raised.  Here is why:

Rails uses a session cookie as opposed to a persistent cookie to store the *authenticity token*.  By design, session cookies are deleted when browsers are closed.  Upon re-opening mobile Safari, the browser tab is reloaded entirely from cache and the session cookie is gone.  Thus, upon submitting the login form the `authenticity_token` in the form cannot be verified.  This didn’t seem to be an issue with other browsers because they appear to retain the session cookies to let users continue where they left off without issue.

### The Solution

Tell Safari (and all browsers) to request the page before submitting the again.  This can be done via the `Cache-Control` header and these two directives: `no-cache` and `no-store`.

* `no-store`: a response is not allowed to be cached by the browser. It must be fetched on every request.
* `no-cache`: caches need to submit the request for validation first to check if the resource has changed.

{% highlight ruby linenos %}
# config/application.rb
class Application < Rails::Application
    config.action_dispatch.default_headers.merge!(
      'Cache-Control' => 'no-store, no-cache'
    )
end
{% endhighlight %}

Actually, I don’t know why both are needed.  It seems to me that `no-store` should be enough.  

After deploying the above change and monitoring the app for a week, I haven’t seen a single `ActionController::InvalidAuthenticityToken` exception.  For more information on the topics covered in this post, see resources below.
 

### Resources

* [Ruby on Rails security guide][rails security]
* [Rails forgery protection][rails forgery protection]
* [Cache-Control header spec][cache-control moz]
* [HTTP Caching tutorial by Ilya Grigorik][google caching]

[trim agency]: http://trimagency.com/
[gh issue]: https://github.com/rails/rails/issues/21948
[csrf wiki]: https://en.wikipedia.org/wiki/Cross-site_request_forgery
[rails forgery protection]: http://api.rubyonrails.org/classes/ActionController/RequestForgeryProtection/ClassMethods.html
[cache-control moz]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control
[google caching]: https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching?hl=en#cache-control
[rails security]: http://guides.rubyonrails.org/security.html#csrf-countermeasures


