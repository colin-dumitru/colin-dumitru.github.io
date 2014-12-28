---
layout: post
title:  "Functional operations in ES6"
date:   2014-12-28 19:00:00
categories: functional-programming JavaScript tutorial
---

There's no denying a lot of cool stuff will be coming in ES6. Generators,
classes, modules, proxies, and a lot more goodies which will bring JavaScript into
the modern age of programming. But for this short Blog post, we'll be focusing only
on the functional aspect of the upcoming ES6 changes.

The goal is to rewrite common operations available natively in other
functional programming languages into JavaScript (with mostly one liner expressions).

All these examples have been tested on [Firefox 34](https://www.mozilla.org/en-US/firefox/new/),
which includes implementations for all the upcoming ES6 features used in this
post.

# Arrow functions

Although not strictly linked to functional programming, we will use the
"arrow" notation extensively in this post, and it's best if we get this syntax
out of the way.

**syntax**
{% highlight text %}
(<param>[, <param>...]) => {
  <statements>
}
{% endhighlight %}

Or if the function contains a single parameter, the parenthesis can be omitted.
{% highlight text %}
<param> => {
  <statements>
}
{% endhighlight %}

You can go even shorter if there's a single statement inside the function block.
In this case, the return statement can be inferred.
{% highlight text %}
<param> => <statement>
{% endhighlight %}

And to top it off, here are some examples with their equivalent arrow conversion.

{% highlight JavaScript %}
// This example
[1, 2, 3].reduce(function(x, y) { return x + y });
// Is quivalent to
[1, 2, 3].reduce((x, y) => x + y);

// And this example
["John", "Mike", "Kim"].map(function(x) { return {name: x}});
// Is quivalent to
["John", "Mike", "Kim"].map(x => ({name: x}));
{% endhighlight %}

Note the parentesis around the returned object for the last example.
 This is to prevent ambiguity between a function body and declaring a new object.

But it's not just syntactic sugar, there's also a very important functional difference
between regular functions and Arrow functions: *lexical "this"*. To put it
plainly, "this" is always assigned to the owning object, and not the calling
object.

Take this example.

{% highlight JavaScript %}
function Person(){
  this.name = "John";

  this.getName = function() {
    return this.name;
  }

  this.getActualName = () => this.name;
}

var p = new Person(),
    getName = p.getName,
    getActualName = p.getActualName;

console.log(getName()) //output undefined
console.log(getActualName()); //output "John"
{% endhighlight %}

With regular functions declared on objects, the "this" keyword always
referes to the calling object. That's why, for the first "console.log", the
output is "undefined" and not "John"; the "this" reference points towards the
"window" object and not the "p" object. For the second "console.log", the
"getActualName" function remembers who is it's "this" reference (in our case "p"),
and the output is "John", as any sane programming language should output.

# For comprehension

Unlike Arrow functions, the new "For" syntax introduced with ES6 is just
syntactic sugar over function calls. The "for" statement can be replaced with
a "foreach" method call, and the "if" statement with a method call to "filter".

**syntax**
{% highlight text %}
[ for (<variable> of <iterable>) | if (<condition>)... <statement> ]
{% endhighlight %}

You can put an unlimited number of "for" and "if" statements, although the
expression must always start with "for".

**example**
{% highlight JavaScript %}
// This example
[ for (i of [1, 2, 3, 4, 5]) if (i % 2 == 0) 3 * i]
// Is equivalent to
[1, 2, 3, 4, 5].filter(i => i % 2 == 0).map(i => 3 * i)

// This example
[ for (i of ["John", "Mike"]) for (j of [1, 2]) i + j]
// Is equivalent to
var names = ["John", "Mike"],
  orders = [1,2],
  result = [];

for (var i = 0; i < names.length; i++) {
  for (var j = 0; j < orders.length; j++) {
    result.push(names[i] + orders[j])
  }
}
console.log(result);
// And outputs [ "John1", "John2", "Mike1", "Mike2" ]
{% endhighlight %}

Quite the difference, isn't it? We'll use the new "For" syntax extensively in
this blog post, so I'll give more examples as we go along.

# Map

This brings us to our first operation, "Map". This isn't a new concept
in JavaScript, so I won't go into much detail. This operation takes a
method as input which is applied to each member of an input iterrable. The
result of each method call is grouped into another Array, which is
given as output.

We can either use the regular "map" function to write this operation, or
the "for" comprehension statement.

**example**
{% highlight JavaScript %}
var names = [ "John", "Mike", "Colin" ];

// This statement
names.map(n => n.length);
// Is equivalent to
[for (n of names) n.length];
// And outputs [ 4, 4, 5 ]
{% endhighlight %}

The example takes as input a list of strings, and converts each member of the
array to the length the input string.

# Filter

Another basic operation which has been in JavaScript for quite some time.
It involves "filtering" or "selecting" an input iterrable based on a predicate
(a function which receives as input a single element from the given array, and
outputs a boolean value). If the return value is true, then the element
gets to stay in the array, otherwise it is discarded.

This operation can be written with the new "for" syntax or with the "filter"
method declared on a native array.

**example**
{% highlight JavaScript %}
var numbers = [1, 2, 3, 4, 5];

// This statement
numbers.filter(n => n % 2 == 0);
// Is equivalent to
[for (n of numbers) if (n % 2 == 0) n];
// And outputs [ 2, 4 ]
{% endhighlight %}

This examples takes as input an array of numbers, and outputs only those which
are even.

# Zip

This operation aggregates two separate lists, into a single list of pairs. It puts
the first element of both input arrays input a single output pair, the second
elements of both input arrays into another pair, and so on. It's best understood
with an example.

Unfortunately, ES6 does not define a "zip" method, so we'll have to use
an alternate syntax to achieve the same result.

**example**
{% highlight JavaScript %}
var numbers = [1, 2, 3],
    names = ["John", "Mike", "Colin"];

// This statement
numbers.map((n, index) => [n, names[index]])
// Outputs [[1,"John"],[2,"Mike"],[3,"Colin"]]
{% endhighlight %}

We'll use the fact that the second argument to the "map" callback is the
elements index. Each element of the output array is an array of length 2, where
the first element is from the first array, and the second element is from the
second array.

# Partition

This operation splits an input array using a boolean function, which is applied
on each member of the original array. If the method returns true, then the element
is put into the first list, otherwise it is put into the second one.

This operation also doesn't have an equivalent in JavaScript, but it is possible
to write a one-liner using the reduce function.

**example**
{% highlight JavaScript %}
var input = [1, 2, "John", 3, "Mike", "Colin"],
  // This predicate separates strings from numbers.
  p = x => x.length;


// This statement
input.reduce(
  (l, r) => ( (p(r) ? l[0] : l[1]).push(r),  l ),
  [[],[]]
);
// Outputs [["John","Mike","Colin"],[1,2,3]]
{% endhighlight %}

I apologise for this.

We split the input array into two separate ones: numbers and strings, by using
a reduce operation which converts our input array into a single pair.

The second argument to the reduce operation is the starting value, which we set
to be a pair or two empty arrays. This way, the "l" (or left) value of the reduce
operation will always by our initial pair, in which we push elements based on
where they fall with respect to our "p" predicate.

To make thinks clearer, here is the non "oneliners make the program run faster"
version:

{% highlight JavaScript %}
var input = [1, 2, "John", 3, "Mike", "Colin"],
  p = x => x.length;

input.reduce(
  (l, r) => {
    if (p(r)) {
      l[0].push(r);
    } else {
      l[1].push(r);
    };
    return l;
  },
  [[],[]]
);
// Outputs [["John","Mike","Colin"],[1,2,3]]
{% endhighlight %}

So plain, blah.

# Find

This operation finds the first element from an input iterrable which matches
the given predicate.

Thankfully, we have a direct equivalent into JavaScript.

{% highlight JavaScript %}
var names = ["Colin", "Mike", "John"];

// This statement
names.find(x => x.length == 4);
// Outputs "Mike"
{% endhighlight %}

We search for the first string element, whose length is 4, which in our case is
"Mike". That's all there is to this operation.

# Drop

As the name implies, this operation "drops" the first <n> elements from an input
array.

We also have a direct equivalent for this operation ins ES6, quite a few
actually. But we'll be using the "slice" method, which returns a
shallow copy of the original array.

{% highlight JavaScript %}
var numbers = [1, 2, 3, 4, 5];

// This statement
numbers.slice(3);
// Outputs [4, 5]
{% endhighlight %}

# Drop While

Similar to the "Drop" operation, but instead of dropping a fixed number of
elements, we remove the first elements which match a given predicate.

We don't have a direct correspondent into ES6, so we'll be using two method
calls to achieve the same effect: ***slice*** (which we covered above) and
***findIndex*** which returns the index of the first element which matches
the given predicate.

We must also reverse the predicate, so the ***findIndex*** method returns
the first element which does *not* match our initial predicate.

{% highlight JavaScript %}
var numbers = [2, 4, 6, 1, 2, 3],
    // This preducate returns true for even elements.
    p = x => x % 2 == 0;

// This statement
numbers.slice(numbers.findIndex(x => !p(x)))
// Outputs [1, 2, 3]
{% endhighlight %}

In this example, we drop the first elements which are even.

# Fold Left

This operation reduces an entire iterrable to a single value, by continuously
applying a binary function on all elements. It's no wonder that the equivalent
method in JavaScript is called "reduce".

This method takes two forms, depending on the number of parameters which are
passed. If only a single parameter is given (the reduce function), then this
method first applies the binary function on the first two elements of the array,
and the resulting value is then passed as the "right" parameter to subsequent
calls.

If a second argument is given (the starting value) then the first binary
call is made with the first element of the array and the second argument
to the "reduce" function.

{% highlight JavaScript %}
// This statement
[1, 2, 3, 4, 5].reduce((l, r) => l + r);
// Outputs 15

// And this statement
[1, 2, 3, 4, 5].reduce((l, r) => l + r, 20);
// Outputs 35
{% endhighlight %}

# Fold Right

Similar to the previous operation, except the binary function is applied
from right to left.

{% highlight JavaScript %}
// This statement
[1, 2, 3, 4, 5].reduceRight((l, r) => (l.push(r), l), [])
// Outputs [5, 4, 3, 2, 1]
{% endhighlight %}

We reverse the initial array, by consecutively pushing all of its elements into
a new array from right to left.

# Flatten

The "Flatten" operation concatenates a list of arrays into a single list.
Unfortunately, ES6 doesn't provide a direct correspondent, so we'll have
to use the "apply" hack on a list of arrays.

{% highlight JavaScript %}
// This statement
Array.prototype.concat.apply([], [[1, 2, 3], [4, 5]])
// Outputs [1, 2, 3, 4, 5]
{% endhighlight %}

We apply the "concat" function on each element from our input array, resulting
in a single output array with all the elements from all our original arrays.

# Final Words

These are the most common functional operations I personally use on arrays in
 JavaScript (or any other language with functional asspects). I know my way of
 writing them might not be optimal, but it does get the job done.

**References**

+ [MDN Array comprehensions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Array_comprehensions)
+ [Arrow Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
