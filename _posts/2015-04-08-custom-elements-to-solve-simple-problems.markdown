---
layout: semantic
title:  "Using Custom Elements to solve simple problems"
date:   2015-04-08 12:53:09
categories: semantic-ui, custom-elements, ampersand, backbone
---

[Webcomponents](http://webcomponents.org/) always seem just over the horizon, really cool but [not quite ready to use in production.](http://developer.telerik.com/featured/web-components-arent-ready-production-yet/)
 I'm starting to worry that the full spec never will, as we continue to get nothing but silence from IE and Safari. 
However, I'm here to tell you about something that *does* seem ready to go today, Custom Elements.  Unlike the claims of Web Components, 
I'm not going to tell you that Custom Elements are going to revolutionize your development life and change the web, but 
I am going to hopefully show you how they can be a useful tool in your development toolbox.


## Some Background
Custom Elements are an important part of the webcomponents spec, letting you define (as the name implies), Custom Elements.  There is an 
[excellent introduction article](http://www.html5rocks.com/en/tutorials/webcomponents/customelements/) to them, and I encourage you to read it 
to get a good overview of them.  There is also an article similar to this one [talking about production ready Custom Elements](http://developer.telerik.com/featured/web-components-ready-production/). 
I want to expand on that and show you some useful examples of using them today.

The whole point of making your own Custom Elements, beyond making your HTML more semantic, is that you can add your own 
behavior or methods to those elements.  This can be useful, as long as we don't try to do too much.  

### A more specific example, Dropdown components 
Many people use [Bootstrap](http://getbootstrap.com/) for their CSS library, we're using [Semantic-UI](http://semantic-ui.com/).
I'll be focusing on Semantic-UI but this example is applicable for Bootstrap.   Whatever you use, it probably has some objects that are a mix of css and javascript.  These are called "components" in Semantic-ui and "plugins" in Bootstrap. 
Basically to get the nice behavior and styling you want together, with the caveat that
 you usually have to include not only the CSS but also a javascript file.  These
libraries usually scope their modules as a jQuery plugin so you end up doing something like :
{% highlight html %}
<!-- Your HTML usually looks something like this -->
<div class="dropdown">
    <!-- inner dropdown contents -->
</div>
{% endhighlight %}
And your javascript ends up looking like this:
{% highlight javascript %}
$('.dropdown').dropdown()
{% endhighlight %}

And you get a nice dropdown:

{% include custom-elements/basic-dropdown.html %}

&nbsp;

This is usually not a big deal, but it is a source of some common mistakes.  Initializing your dropdown forces you to 
get more involved in the lifecycle of the HTML.  If you have a static site, then you can just do a global `$('.dropdown')` 
selector and be good, but if you have dynamic HTML you must initialize a dropdown every time you insert one in the DOM. 
I often forget to put the `.dropdown()` code in when I'm creating a new view and then it doesn't work, causing some debugging problems.  
Wouldn't it be cool if I could just say it was a dropdown and be done with it?  Something like:

{% highlight html %}
<semantic-dropdown class="ui dropdown">
    <!-- inner dropdown contents -->
</semantic-dropdown>
{% endhighlight %}

With Custom Elements, you can.  

### Building a Custom dropdown element

Building a Custom Element is surpisingly easy!  You just create a prototype object for your element, usually inheriting from `HTMLElement`
and then call `registerElement` on the document with the name of the element and the prototype.   Here's our Custom dropdown:
{% highlight javascript %}
var proto = Object.create(HTMLElement.prototype);

proto.attachedCallback = function(){
    $(this).dropdown();
};

return document.registerElement('semantic-dropdown', {
    prototype: proto
});
{% endhighlight %}

You can wrap this code in the module system of your choice (AMD or CommonJS), evaluate it and you're ready to go!  Now, anytime you insert
 a `semantic-dropdown` in your DOM, it automatically initializes!  The `attachedCallback` 
 function will be called anytime your Custom Element is inserted into the DOM.  There are other lifecycle methods you can hook into, `createdCallback`, `attachedCallback`, and `attributeChangedCallback` which allows you to do any teardown you need (which for dropdowns is nothing). 
 
This doesn't preclude you using the javascript API of Semantic at all though, feel free to call `.dropdown()` with your own custom options
if you like.  If they're simple options, you can even put them in the attributes of your Custom Element and key off of them.  For example, 
Semantic dropdowns have a way to [change their transitions in the initialization from the default](http://semantic-ui.com/modules/dropdown.html#/examples).  
The normal way is to do this: 
{% highlight javascript %}
$('.dropdown')
  .dropdown({
    // you can use any ui transition
    transition: 'drop'
  });
  
{% endhighlight %}
This is easy enough to do in our new Custom Element as well:

{% highlight javascript %}
var proto = Object.create(HTMLElement.prototype);

proto.attachedCallback = function(){
    var transition = this.getAttribute('transition') || "someDefault";
    $(this).dropdown({
        transition : transition
    });
};

return document.registerElement('semantic-dropdown', {
    prototype: proto
});

{% endhighlight %}
{% highlight html %}
<semantic-dropdown class="ui dropdown" transition="drop">
    <!-- inner dropdown contents -->
</semantic-dropdown>
{% endhighlight %}

The level of configuration or defaults you want to provide are up to you.  If you'd like to give users the option to not
put the `ui dropdown` classes on your element, no problem, just add them yourself in the element's created callback.  

### Adding methods
You can also add your own custom methods to your elements.  A common one for a dropdown is a `toggle` call.  We can add that
method directly to the prototype of our Custom Element and call it directly.

{% highlight javascript %}
var proto = Object.create(HTMLElement.prototype);

proto.attachedCallback = function(){
    $(this).dropdown();
};

proto.toggle = function(){
    $(this).dropdown('toggle');
};

return document.registerElement('semantic-dropdown', {
    prototype: proto
});
{% endhighlight %}

Then, when we're using our dropdown, we don't need to wrap it up in a `.dropdown()` call, we can just call `toggle()`.  

{% highlight javascript %}
// Careful this won't work and will call jQuery's toggle method!
$('semantic-dropdown').toggle(); 
// this is how you want to do it
$('semantic-dropdown')[0].toggle();
// or leave jQuery out of it altogether
document.querySelector('semantic-dropdown').toggle();
{% endhighlight %}

Notice the comments above.  jQuery doesn't have an easy way to invoke custom methods on elements because IMO there aren't that 
many interesting ones to invoke.  

Put it all together and you can get a working custom dropdown like the one below.  Go ahead and open that inspector on the element below. 
While you're at it, go ahead and evaluate this piece of code in the inspector:  `document.querySelector('semantic-dropdown').toggle();`
&nbsp;

{% include custom-elements/custom-dropdown.html %}

&nbsp;


### And so on. 
This is relatively easy.  Is it earth-shattering?  No, but is it a nice tool to have in your toolbox?  Definitely.  
It would be awesome if some of the CSS frameworks we use started to embrace some concepts like this and move some default 
 module behavior into custom elements. 

## Another example:  Using an element to do some application logic
I do a lot of Backbone / [Ampersand.js](http://ampersandjs.com/) / Marionette development and really like [Backbone Radio](https://github.com/marionettejs/backbone.radio).
  For things like menus, we use Radio commands and the ampersand convention of 'data-hook' attribtues to keep our styling separate from our javascript behavior.  In the end it ends up something like this: 
    
{% highlight html %}
<div class="ui blue pointer menu" >
    <div class="active item" data-hook="tab-1">Tab1</div>
    <div class="item" data-hook="tab-2">Tab2</div>
    <div class="item" data-hook="tab-3">Tab4</div>
    <div class="item" data-hook="tab-4">Tab5</div>
</div>    
{% endhighlight %}

And then we hook up those up in a View: 
{% highlight javascript %}
 var View =  Ampersand.View.extend({
    template: template,
    events : {
        'click [data-hook=tab-1]' : 'loadTab1',
        'click [data-hook=tab-2]' : 'loadTab2',
        'click [data-hook=tab-3]' : 'loadTab3',
        'click [data-hook=tab-4]' : 'loadTab4'
    },
     loadTab1 : function() {
        this.activate(this.$('[data-hook=tab-1]')); 
        Radio.command('application','show:tab1');
    },
    
    activate : function(){
     // for styling, remove active class on others and add it here
    },
    // and so on.. 
    
{% endhighlight %}

As you can see, this can end up like a lot of boilerplate just to trigger a Radio command on a click.  So we're going to try to write a 
 small element that will help remove a little of this.  

{% highlight javascript %}
 var proto = Object.create(HTMLElement.prototype);

 proto.attachedCallback = function(){
     var el = this;
     this.clickListener = function(){
         var command = el.getAttribute('command');
         var channel = el.getAttribute('channel');
         Radio.command(channel,command);
     };
     this.addEventListener('click', this.clickListener);
 };
 
 proto.detachedCallback = function(){
     this.removeEventListener('click',this.clickListener);
 };

 return document.registerElement('radio-command', {
     prototype: proto
 });
{% endhighlight %}

Fairly straightforward.  We create a clickListener which grabs the `command` and `channel` attributes and trigger a 
Radio command on it via a click.  We keep the eventListener on the element so we can be good citizens and remove the
event listener when the element is removed in the `detachedCallback`.  
Now, we can replace our HTML with our new Custom Element, using two attributes to pass in arguments to our element.  
{% highlight javascript %}
<div class="ui blue pointer menu" >
  <radio-command channel="my-channel" command="show:tab1" class="item" >Tab1</radio-command>
  <radio-command channel="my-channel" command="show:tab2" class="item" >Tab2</radio-command>
  <radio-command channel="my-channel" command="show:tab3" class="item" >Tab3</radio-command>
  <radio-command channel="my-channel" command="show:tab4" class="item" >Tab4</radio-command>
</div>    
{% endhighlight %}

### Adding behavior to children

You'll notice a lot of duplication above, they're all using the same radio channel.  It would be cool to pull that channel up 
to the parent div so you do something like this:

{% highlight html %}
<radio-command channel="my-channel" class="ui blue pointer menu" >
  <div command="show:tab1" class="item" >Tab1</div>
  <div command="show:tab2" class="item" >Tab2</div>
  <div command="show:tab3" class="item" >Tab3</div>
  <div command="show:tab4" class="item" >Tab4</div>
</radio-command>    
{% endhighlight %}

The code to do this with a Custom Element would look something like this:

{% highlight javascript %}

var proto = Object.create(HTMLElement.prototype);

proto.attachedCallback = function(){
    var el = this;
    var channel = el.getAttribute('channel');
    this.clickListener = function(event){
        var command = event.target.getAttribute('command');
        if(command){
           Radio.command(channel,command);
        }
    };
    el.addEventListener('click', this.clickListener);

};
proto.detachedCallback = function(){
    this.removeEventListener('click',this.clickListener);
};

return document.registerElement('radio-channel', {
    prototype: proto
});
{% endhighlight %}
The main difference here with above is that we're using event bubbling to catch events from children and using the `event.target` 
 get the clicked-on node, grab the attributes from there and use them to fire off our Radio command.
Again, notice how we're using the `detachedCallback` to be good citizens and unregister our listener when the elements are removed.

## What Custom Elements are not
Custom Elements are often lumped together with the other specs of Web Components (Shadow DOM, HTML Imports, and Templates) when 
people talk about Web Components, and rightfully so, the power of them all working together does indeed promise to change
the way we develop web applications.  

Custom Elements do not allow you to easily build components with their own markup and custom styles.  Sure you can use `.innerHTML` and put some 
DOM in there, but its not encapsulated and isolated from other parts of your page.  This is what Shadow DOM does and its the 
coolest part of Web Components, but also the least supported by certain browsers, and hardest to Polyfill.    

## Are they really ready for production?

This is the big question about webcomponents and usually the answer is "not yet" but Custom Elements are a [different story](http://developer.telerik.com/featured/web-components-ready-production/).  Github 
[has been using Custom Elements](http://webcomponents.org/articles/interview-with-joshua-peek/)
in production for some time now and according to the article, the reason is the polyfill is small, simple and covers most edge cases, which is 
much harder to do for a more bleeding edge technology like shadow DOM.  The polyfill is [unfortunately still required](http://caniuse.com/#feat=custom-elements) but
is relatively small (only about 10k minified)

## Conclusion
I hope I have shown you a new tool for your development toolbox.  To me, it seems we get really excited about the full 
web components spec and are let down when we find out it's not quite ready to go.  So we end up not using them at all, but
Custom Elements are much more mature and can be used today to easily solve some problems.   

[jekyll]:      http://jekyllrb.com
