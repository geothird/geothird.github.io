---
layout: post
title: Ruby Blocks
category: posts
---

I love using Ruby code [blocks][blocks].
They seem like magic at first glance, but then you see how powerful and fun
they are.

{% highlight ruby %}
#!/usr/bin/ruby
def invisible_args
	yield rand(10)
end
invisible_args { |x| puts "#{x}" }
{% endhighlight %}
<br/>
Output:

{% highlight bash %}
$ ruby blocks.rb
8
{% endhighlight %}
<br/>

This method has no arguments listed on the invisible_args method.  It executes the block with a parameter of `rand(10)` in this case that yeilds us `8`.  

---

We can however have a block as a parameter and use that block a bit differently.
Take this method for example:


{% highlight ruby %}
#!/usr/bin/ruby

def proc_args(&block)
  if block_given?
    yield *(0..block.arity)
  else
    puts 'No block given.'
  end
end

proc_args { |x,y,z| puts "#{x}, #{y}, #{z}" }
{% endhighlight %}
<br/>
Here we are taking a block as a parameter, this will convert the block into a [Proc][proc] object and allow us to use the [arity][arity] method on it which `returns the number of arguments`.  Using a range we create 0 through argument length, and coerce it into an array using the splat operator.

The result is that this block is then called an incremented number as each argument.

{% highlight bash %}
$ ruby blocks.rb
0, 1, 2
{% endhighlight %}
<br/>

In this case `yeild` and `block.call` are interchangable use whichever you prefer. 

---

Currently these examples assume no type, and no limit of parameterrs essentially [Duck typed][duck]. 

Lets see what happens when we change the block to using a different type.


{% highlight ruby linenos %}
proc_args { |x,y,z| puts "#{x + y + z}" }
{% endhighlight %}
<br/>
Output:

{% highlight bash %}
$ ruby blocks.rb
3
{% endhighlight %}
<br/>

So far so good, putting the arguments as a string worked, and now summing the list of numbers worked.  This was done without changing the method at all just the block that was passed in.

The only problem is that this method really doesn't do anything. The same thing could be more easily accomplished by just calling a range. 

This was a discussion of how blocks can be used, coming up with something useful is where imagination comes in, this is omething you can put in when you have a problem that needs solved and blocks pop up as a possible solution.

[blocks]: http://www.ruby-doc.org/core-2.1.1/Proc.html
[proc]: http://www.ruby-doc.org/core-2.1.1/Proc.html
[arity]: http://www.ruby-doc.org/core-2.1.1/Proc.html#method-i-arity
[duck]: http://en.wikipedia.org/wiki/Duck_typing
