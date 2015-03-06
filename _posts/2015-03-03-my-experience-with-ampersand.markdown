---
layout: post
title:  "Upgrading your Backbone with Ampersand.js"
date:   2015-03-03 12:53:09
categories: ampersand, Marionette, Backbone 
---

The last several months have seen an explosion of javascript frameworks.  I'm here to tell you about one of those, Ampersand.js,
my experience with it, and how it has worked.


## From Backbone
Ampersand is basically a fork of Backbone, split up into small modules and libraries and augmented with some nice
features you always wanted, so if you love Backbone's flexibility, you'll feel right at home with ampersand.  In fact,
this is actually one of the best features of ampersand, which I'll talk about later.  

##Backbone.Model -> Ampersand State
View layer technologies like React are very plentiful around the web, and these libraries are 
always saying "it's basically the view layer", so what about the other layers?  It's easy to use Backbone for these 
view layers, but Backbone models have a special syntax for setting their properties, the `get` or `set` methods.  This
is great if you stay in the Backbone/Marionette world, but React views don't want to call `props.get('property')` to get their 
property, they want to call `props.property`.  They're expecting that you are passing around plain old objects or classes 
that you access like every other property.  

### Enter Ampersand State
To me, the biggest thing about ampersand state that I just *love* is that they decided to embrace property definition to 
give you an API and evented properties that you have come to know and love from Backbone but without the `.get` syntax. (though you can still use that syntax)
Admittedly, to do so, they have decided to abandon IE8 support, and that's a caveat you'll have to consider. 

####Backbone Model definition
{% highlight javascript %}
/*  Define properties here as a good practice
**  Name : string
**  Done : boolean
*/
var TODO = Backbone.Model.extend({

});
var todo = new TODO({ name : 'have fun', done : false});
todo.get('name') // => have fun
todo.on('change:name', function(model){
   // model name changed
});
{% endhighlight %}


####Ampersand State definition
{% highlight javascript %}
var TODO = AmpersandState.extend({
  props : {
    name : 'string',
    done : 'boolean'
  }
});
var todo = new TODO({ name : 'have fun', done : false});
todo.get('name') // => have fun
todo.name // => have fun!
todo.on('change:name', function(model){
   // model name changed
});
{% endhighlight %}

Don't worry if you don't like defining every property beforehand.  Ampersand lets you allow 'extraProperties' to exist and 
be dynamically created on the fly if you want that flexibility.

Ampersand State also has derived properties, which can be very useful.  These are cached so you can build higher level properties 
on top of your core properties and drive your views and app off of them.  We'll see more of this below.

Finally the kicker for me was the presence of default child models and collections.  We have complex data models 
on our projects, several levels deep, and the ability to just specify a nested json object as a child with a model or 
a collection with a collection of models helped us finally ditch Backbone Relational in favor of this simpler approach.  

I haven't even gotten into the Ampersand views yet.  Suffice it to say they are to Backbone views as models are to ampersand state. 
I'll probably have a separate blog post about them.  

#### Example "Complex" state object
{% highlight javascript %}
var Thread = AmpersandState.extend({
  modelType : 'Thread'
  props : {
    id : 'number',
    name : {
        type : 'number',
        required : true,
        default : 'default Thread'
    },
    collections : {
       tasks : Task.Collection // defined elsewhere
    },
    children : {
       address : Address.Model // defined elsewhere
    },
    derived : {
       initials : {
         deps : ['name'],
         fn : function(){
            // grab first letter of each word and capitalize
            // ignore lodash craziness if you want
            // 'default Thread' -> D T
             return _(this.name.split(' ')).map(function(word){
                 return word[0];
             }).join(' ').value();
         } 
       }
    }
  }
});
{% endhighlight %}

The cool thing about derived properties is they are cached and only trigger if the property itself changes.  For example, if I changed the name in the State above to "Def Trunk", the initials property would not change and no change event would get triggered.  Pretty cool. 

##It just works

The other awesome thing about Ampersand comes from its closeness to the Backbone world.  The [View](http://ampersandjs.com/docs#ampersand-view) in Ampersand follows some simple [conventions](http://ampersandjs.com/learn/view-conventions) which also happens to match almost all Backbone / Marionette views.  This is where things get really awesome.  A Marionette view or Backbone View can use Ampersand Models and State as their models, they don't care.  Also, Marionette views can render and show Ampersand Views just as easily.  We like Marionette Layouts in our project, and they can easily show Ampersand Views because they follow the same conventions.  It just works!  

## Progressive Enhancement

So we took the plunge and have been very happy with the results on our project.  The community is great and helpful and we've been doing new develoment with Ampersand objects, upgrading old Backbone/Marionette views as we go if we need to.  

So I encourage you to check out Ampersand.js.  The modules are all tiny and composable so you can just take what you want.  Thanks for reading!


[jekyll]:      http://jekyllrb.com
