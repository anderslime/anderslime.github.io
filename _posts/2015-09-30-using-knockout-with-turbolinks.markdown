---
layout: post
title:  "Using Knockout.js with Turbolinks"
description: Code snippet for making Knockout.js work with Turbolinks
date: 2015-09-30
---

I'm currently working on a project that use
[Turbolinks](https://github.com/rails/turbolinks) and
[KnockoutJS](https://knockoutjs.com) together, which gave some initial
problems since Turbolinks distrupts the normal web application load cycle.

## TL;DR
To avoid trouble when using knockout with Turbolinks i replaced the normal
*ko.applyBindings* with

{% highlight javascript %}
window.applyTurboBindings = function(node, viewModelAccessor) {
  return $(document).one("page:change", function() {
    if (!(node && ko.dataFor(node))) {
      return ko.applyBindings(viewModelAccessor(), node);
    }
  });
};
{% endhighlight %}

that can be used with:

{% highlight javascript %}
applyTurboBindings(document.getElementById('my-node'), function() {
  return new MyViewModel();
});
{% endhighlight %}


## So what is Turbolinks?

We use Turbolinks in the project to give a better user experience by making
the web application feel faster. Turbolinks replace the body (or parts) of the
DOM similar to
[pjax](https://github.com/defunkt/jquery-pjax), which means that the browser
does not have to recompile the JavasScript. One caveat though is that a lot
of the jQuery events does not work as you expect. One example is the
`ready`-event, which is normally fired everytime you go to a new page, but
with Turbolinks this event is only fired on the initial page load and not
when the user navigates to another page. Instead Turbolinks provides its
own custom events fired at different points of a page lifecycle.

## And what about Knockout.js?

[Knockout.js](http://knockoutjs.com/) is a JavaScript library that provides
declarative bindings between the DOM and your model and more recently also
supports
[web components](http://knockoutjs.com/documentation/component-overview.html).
To be more specific it makes it possible to define view models that holds
and decorates the data and behaviour, which are then bound (through `data-bind`
HTML declarations) to the DOM such that the DOM automatically updates based
on the behaviour and data in your view model.

## So what is the problem?

The normal flow for setting up knockout bindings is that you have a DOM node
that contains the `data-bind`-declarations and your view model class or
function on which you call `ko.applyBindings` as following:

{% highlight html %}
<div id="my-node">
  <input data-bind="value: catName" />
  <div>Your cat's name is: <span data-bind="text: catName"></span></div>
</div>

<script type="text/javascript">
  function CatViewModel() {
    this.catName = ko.observable("Mr. Tickles");
  }
  $(document).ready(function() {
    var node = document.getElementById("my-node");
    ko.applyBindings(new CatViewModel(), node);
  });
</script>
{% endhighlight %}

This code is a simple input field and an element that tell's what your cat's
name is based on what you type in the input field. In order to map the
binding of `catName` between the `data-bind`-declarations and the `catName`
property, we use the function ko.applyBindings on the view model and a
surrounding DOM node. This code would probably work on the first page load, but
let's say the above code is fetched and injected using Turbolinks after a
page change at which the `ready`-event is not executed (well, since the
document was already `ready` after the initial page load).

Now! We are lucky that Turbolinks provide events for when we fetch and load
pages through Turbolinks. The most approriate for this purpose is the
`page:change`-event, which is fired after the body has been replaced after a
page change. With this information we try to apply bindings with the code:

{% highlight javascript %}
$(document).on("page:change", function() {
  var node = document.getElementById("my-node");
  ko.applyBindings(new CatViewModel(), node);
});
{% endhighlight %}

Although this seems perfect there is one problem:

{% highlight md %}
Uncaught Error: You cannot apply bindings multiple times to the same element
{% endhighlight %}

Yep, the ko.applyBindings method is not idempotent, which means when you try
to execute it on the same DOM node twice it explodes with this error. And
since you register the function to execute on EVERY *page:change* it also executes
when we fetch the following pages. This means we have two problems:

* We apply bindings on all future *page:change*-events after we loaded the given page.
* If the *page:change* event is executed twice the page explodes
<br>
<br>

## And how do we solve it?
To solve this problem we use [jQuery's one](http://api.jquery.com/one/) to
ensure the event is only triggered once.

This leads to the final solution:

{% highlight javascript %}
window.applyTurboBindings = function(node, viewModelAccessor) {
  return $(document).one("page:change", function() {
    if (!(node && ko.dataFor(node))) {
      return ko.applyBindings(viewModelAccessor(), node);
    }
  });
};
{% endhighlight %}
