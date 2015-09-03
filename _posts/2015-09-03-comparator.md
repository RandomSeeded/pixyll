---
layout:     post
title:      Reverse Sorting Collections in Backbone
date:       2015-09-03
summary:    (Ab)using comparators
categories: Backbone sorting javascript
---

Sorting collections in backbone is easy. Invoke `sort()` and go on your way. But what if we want to change the sort order? No baked in functionality for that, we've got to do it on our own. How? Oh, there are so many ways.

## Code for testing

A little bit of code to make our testing easy:

{% highlight js %}
var Model = Backbone.Model.extend({
  initialize: function(value) {
    this.set('value', value);
  }
});
var Collection = Backbone.Collection.extend({
  model: Model,
  comparator: 'value',
  createArray: function() {
    var results = [];
    this.models.forEach(function(element) {
      results.push(element.get('value'));
    });
    return results;
  }
});

var models = [
  new Model(2),
  new Model(1),
  new Model(3),
  new Model(5),
  new Model(4)
  ];
var collection = new Collection(models);
{% endhighlight %}

We're just creating a few models here and throwing them in a collection, with one additional convenience method `createArray` which we can use to more easily examine our results. So if we now run `collection.sort().createArray()`, we'll get back the array `[1,2,3,4,5]`. Sorted! Great. Let's flip it.

## Method #1:
*the happy way*

As it turns out, comparators don't need to be strings (which represent fields). Instead they can be functions which take a parameter. Then instead of specifying a field which should be used to compare, we instead specify a function which will generate a value used to compare.

{% highlight js %}
comparator: function(element) {
  return -element.get('value');
},
{% endhighlight %}

Wunderbar! Sorted in reverse. But what if our values were strings instead of numbers? Multiplying by negative one wouldn't make much sense...let's change comparator once again.

## Method #2:
*the inflexible way*

{% highlight js %}
comparator: function(a, b) {
  if (b.get('value') > a.get('value')) return 1;
  else return -1;
},
{% endhighlight %}

As it turns out, we can also specify comparator as a function which takes two parameters. Now, instead of specifying a function which returns a single value for a single element, our comparator will be passed two elements and will manually take care of the task of comparing them.

But wait? Wouldn't it be nice if we could easily switch back and forth between our comparator functions? As it is, we'd need to re-define it over and over again each time we wanted to switch.

## Method #3:
*the mildly more flexible way*

{% highlight js %}
comparator: function(a, b) {
  if (a.get('value') > b.get('value')) return 1;
  else return -1;
},

function reverseComparator(comparator) {
  return function(a, b) {
    return comparator(b, a);
  };
}
{% endhighlight %}

Now we only have to define the comparator once. If we want to switch it, all we need to do is pass it in to our reverseComparator function like this:

{% highlight js %}
collection.comparator = reverseComparator(collection.comparator);
{% endhighlight %}

And voila, we have a reversed comparator function. But how will we switch back to the normal version? We *could* run the same line again to re-reverse...but then we're in the ugly position of nesting an ever-increasing number of functions every time we reverse. A better option would be to store a `reversed` flag which simply stores whether or not our function is already reversed.

## Method #4:
*bonus version for masochists*

The above version will work fine, but we did have to both define a custom comparator function, and then create a wrapper function for it. Wouldn't it be nice if we could keep using a standard comparator function?

{% highlight js %}
comparator: function(element) {
  return element.get('value');
}
{% endhighlight %}

Note that the current comparator function isn't handling any reversing; it's just returning the value of whatever field you're interested in. Let's modify it one last time:

{% highlight js %}
function reverseComparator(sortByFunction) {
  return function(left, right) {
    var l = sortByFunction(left);
    var r = sortByFunction(right);

    if (l === void 0) return -1;
    if (r === void 0) return 1;

    return l < r ? 1 : l > r ? -1 : 0;
  };
}
{% endhighlight %}

Now we can initialize comparator to a standard, single-parameter-taking function, and modify it using the reverseComparator function (once again using `collection.comparator = reverseComparator(collection.comparator)`). The reversed version takes two parameters, and so will be passed two elements to compare. It will then *use* the original comparator function provided to generate values for the two elements, and return -1/0/1 based on the results. Credit where credit's due, got this quite cool solution from [here](http://stackoverflow.com/a/12220415/3062972).

# Further reading

For more, I'd recommend checking out the [backbone comparator docs](http://backbonejs.org/#Collection-comparator), as well as [this stack overflow thread](http://stackoverflow.com/questions/5013819/reverse-sort-order-with-backbone-js) on the problem.

