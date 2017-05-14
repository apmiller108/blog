---
layout: post
comments: true
title:  "Setting Default Headers in Angular"
date:   2017-05-13 00:00:00 -0500
categories: angular
---

When building an Angular app, the chances are high that you're going to 
need to set the same headers for most, if not all, of your service 
classes API requests. In this post, I'm going to demonstrate how to set
default headers both statically and dynamically.

### Setting Request Headers in a Service Class ###
For setting headers in a one-off request, we can just define headers as
`RequestOptions` and pass them as a argument to the `http` client's function. Keep in
mind that you'll need to import `Headers` and `RequestOptions` from `@angular/http`
into the service class.

{% highlight typescript linenos %}
@Injectable()
export class WidgetService {

  updateWidget(widget: Widget): Observable<Widget> {
    const headers = new Headers({
      'Content-Type': 'application/vnd.api+json'
    });

    const options = new RequestOptions({
      headers: headers
    });

    return this.http
      .put(`widgets/${widget.id}`, widget, options)
      .map((response: Response) => {
        return response.json();
      });
  }
{% endhighlight %}

Having to do this for every request will get old real fast, not to metion leave you with 
repetitive code. Let's see how to set the `Content-Type` header for all requests by default.

### Setting Static Default Request Headers ###
To ensure that every request from the `http` client is configured with the same headers, we 
need to define a class to hold this configuration. Our class should be a subclass of 
`BaseRequestOptions`.  Then, in the `constructor`, we can set our headers.

{% highlight typescript linenos %}
@Injectable()
export class RequestOptionsService extends BaseRequestOptions {
  constructor() {
    super();
    this.headers.set('Content-Type', 'application/vnd.api+json');
  }
}
{% endhighlight %}

In the `app.module` we can tell Angular to use our custom class instead of 
the default `RequestOptions`.
{% highlight typescript linenos %}
providers: [
    { provide: RequestOptions, useClass: RequestOptionsService }
  ]
{% endhighlight %}

Setting the headers in this way means that the default headers are set when the Angular
application starts up.  In many cases this is just fine, but what if you need to set
headers dynamically?  Let's take a look at how that can be done.

### Setting Dynamic Default Request Headers ###
Let's consider the following use case.  Our app needs to authenticate user requests using an
access token.  The user authenticates with their email and password and the API responds with
a JWT which our app stores in the browser's `localStorage`. Each subsequent request must have
the JWT in the `Authorization` header.  The problem is that before we have added the JWT to
`localStorage`, the headers have already been set.  Our `Authorization` header might look like
this: `"Authorization": "Beaer "` - missing the JWT.  Another use case might involve setting a timestamp. 

The solution is to override the `merge` function.  Every request merges options so we can define 
our dynamic headers there.

{% highlight typescript linenos %}
@Injectable()
export class RequestOptionsService extends BaseRequestOptions {
  constructor() {
    super();
    this.headers.set('Content-Type', 'application/vnd.api+json');
  }
  merge(options?: RequestOptionsArgs): RequestOptions {
    const newOptions = super.merge(options);
    newOptions.headers.set('Authorization',
                           `Beaer ${localStorage.getItem('app-token')}`);
    return newOptions;
  }
}
{% endhighlight %}

### Resources ###
- [RequestOptions documentation](https://angular.io/docs/ts/latest/api/http/index/RequestOptions-class.html)
- [Typescript Classes](https://www.typescriptlang.org/docs/handbook/classes.html)
