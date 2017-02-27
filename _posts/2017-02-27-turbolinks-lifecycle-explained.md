---
layout: post
section-type: post
title: Turbolinks' lifecycle explained
tags: [ 'rails', 'javascript' ]
---

I've spent the last weekend hunting [nasty](https://github.com/reactjs/react-rails/issues/607) [bugs](https://github.com/shakacode/react_on_rails/issues/706) haunting [projects](https://github.com/renchap/webpacker-react) which are trying to marry Rails 5.1 on Turbolinks with ReactJS on Webpack. The reason, why one would want to do it, is a great topic for another post. However, it appears that a misunderstanding of Turbolinks life cycle callbacks has spread widely across our community.

You can read the whole debugging story [in this pull request](https://github.com/renchap/webpacker-react/pull/14#issuecomment-282439136).

It is a long post, feel free to skip to <a href="#tldr">TLDR</a> section.

In this article I will make an attempt to explain how to hook to the Turbolinks' callbacks and properly detect when a page is being shown to the user and when it is being hidden.

When you switch on Turbolinks in your project, the rules of the game change. The javascripts you load will be never unloaded. The side-effects creeping out to `window` or `global` namespace will stay there forever. In return you will get faster page renders and a bit of the Single Page App feeling while still rendering HTML on the server side. Shortly: happy users, sad programmers.

Let's take an example. You go to the foo website, which shows you a fancy, ReactJS-crafted button inviting you to a bar. You click the button, go to the bar, but you realize there is no barista nor other people, and promptly press the back button in your browser. You end up on the previous page. What actually happened?

![Use Case Illustration](/img/2017/02/27/turbolinks-lifecycle-explained/use-case.png)

There were two browser transitions. One led us to the bar, and the other out of it, to the page where the fancy button resides.

### Without Turbolinks

Without Turbolinks, when you click on the button, the whole page is removed from the memory and a new page is requested from the server, parsed, and rendered. After every page load javascripts get a clean state. You don't have to worry about cleaning up your global state or some event listeners. Simple.

### With Turbolinks

Things get interesting, when you start using Turbolinks. You may quickly learn, that you should clean up your mess before moving on to the next page. But what actually happens during the page transitions from our example?

#### Initial visit

First, let's take a look at the initial load of the page, when we visit the site.

![Initial load of the site flow](/img/2017/02/27/turbolinks-lifecycle-explained/initial-load.png)

At the beginning there is nothing: no page, no javascripts, no turbolinks. When the browser gets the HTML and starts evaluating javascripts, Turbolinks gets loaded and emits `turbolinks:load` event. This is when you can initialize your things, but only once! Later, you may want to ignore this event (keep reading to learn, why). For example:

```javascript
document.addEventListener('turbolinks:load', this.mountReactComponents(), { once: true })
```
&nbsp;

#### Visiting an uncached page (transition 1)

When the user clicks the fancy *Go to BAR* button for the first time, Turbolinks will alter the normal HTTP request into an AJAX background call, fire up `turbolinks:before-cache` event, and put the current page into its cache. After the AJAX call finishes, it will fire `turbolinks:before-render` event, replace the `body` and `header` tags' content on the current page with the tags' content from the response, and fire `turbolinks:render` and `turbolinks:load` events.

![Visiting an uncached page flow](/img/2017/02/27/turbolinks-lifecycle-explained/visit-uncached-page.png)

Now, before the new page is rendered, Turbolinks fires up two events: before and after caching. Which one to choose? Ideally, when the user comes back to the cached version of the page, he'd see everything as before he'd left it. The teardown code, like unmounting ReactJS components, may modify the DOM. That's why you should choose the `turbolinks:before-render` event for your teardown code:

```javascript
document.addEventListener('turbolinks:before-render', this.unmountReactComponents())
```

But hey, we don't listen for the `turbolinks:load` event anymore! What should be used for the setup instead?

```javascript
document.addEventListener('turbolinks:render', this.mountReactComponents())
```

&nbsp;

#### Visiting a cached page (transition 2)

The whole fun starts, when the user visits a previously cached page. In that case, Turbolinks will emit `turbolinks:before-render` immediately after caching up the previous page. Then it will replace the `body` and `header` tags with the cached version of the page which is being loaded from the server. This operation is concluded with single `turbolinks:render` event, **without** firing the load event. When the AJAX call is finished, another `turbolinks:before-render` event is emmited (for the cached version), and the flow continues as in the uncached version. The load event is called only after rendering the server version of the page.

![Visiting aa cached page flow](/img/2017/02/27/turbolinks-lifecycle-explained/visit-cached-page.png)

To summarize:

* there is one pair of `turbolinks:before-cache` and `turbolinks:load` events
* there are **two** pairs of `turbolinks:before-render` and `turbolinks:render` events - one pair for a cached version of the target page and the other for a fresh version from the server.

By hooking the setup code to `turbolinks:render` and the teardown code to `turbolinks:before-render` events you can make sure that a cached version of the site is fully functioning and that your teardown code won't be called without a setup before.

### How to improve?

I have to admit it's been a bit confusing at the beginning and clearly it's not only me, given the bugs in the libraries we use. 

Turbolinks could be improved if the `turbolinks:render` event would be fired before firing the load event during the initial load of the page. This move would assign new meanings to these two events:


* `turbolinks:render` would be fired every time a cached or uncached page is displayed
* `turbolinks:load` would be fired only after a fresh page from the server is displayed

<a name="tldr"></a>

### TLDR

If your setup/teardown code has side effects on DOM, use the following snippet:

```javascript
document.addEventListener('turbolinks:load', this.setup(), {once: true})
document.addEventListener('turbolinks:render', this.setup())
document.addEventListener('turbolinks:before-render', this.teardown())
```

If your setup/teardown code does not cause side effects on DOM **and** you don't mind that the cached versions of the pages may be not interactive (just a visual), use the following snippet:

```javascript
document.addEventListener('turbolinks:load', this.setup())
document.addEventListener('turbolinks:before-cache', this.teardown())
```
