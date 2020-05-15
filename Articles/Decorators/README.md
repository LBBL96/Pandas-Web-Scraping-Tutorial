# How to Write and Use Decorators in Python
## @ isn't just for email anymore

![sparkler](lights-dzenina-lukac.jpeg)
### What's a Decorator?
In Python, a decorator is a function that wraps around another function. Its purpose is to provide additional functionality to the function that it wraps.

Let's look at an example.

Say you've written a simple function that adds up two numbers. You also put a doc string in there because you're awesome and want to help anyone who uses your function to understand what it does.

    def add(a, b):
    """
    Takes two parameters, a and b, and returns their sum.
    """
    return a + b


<pre>add(1, 3)</pre>

<pre><b>[Out]</b> 4</pre>

I want to see what your function does, so I call help on it. 

    help(add)

<pre><b>[Out]</b> Help on function add in module __main__:

add(a, b)
    Takes two parameters, a and b, and returns their sum.</pre>

Thank you for that!

Good documentation is essential to good coding. I'll show you an interesting use case in a minute. 

### Let's Write a Decorator

An easy and common decorator is a performance timer. Conveniently, Python includes a performance counter in its built-in library. You'll need to import it.

    from time import perf_counter

If you wanted to see the performance of your function `add()`, you might do it this way:

    start = perf_counter()
    print(add(1, 3))
    end = perf_counter()
    print(f'Run time: {(end - start):0.8f}')

<pre><b>[Out]</b> 4
Run time: 0.00011559</pre>

That was pretty fast. Also, I made the printout nicer-looking by using [f-string formatting](https://docs.python.org/3/library/string.html), with some extra formatting at the end to take the decimals to 8 places.

Notice how `start` and `end` wrap around the `add()` function. This layout makes it a good candidate for a decorator, also known as a wrapper. 

We write a decorator just like any other function. Decorators take a function as their parameter. For this example, I'll call the parameter `fn` to remind me that it's a function. I'll also import `perf_counter` within the function so that I don't have to remember to do that if I use `timer()` elsewhere in the future.

    def timer(fn):
    """
    This is a decorator that returns the time it takes another function to run.
    """
    from time import perf_counter

    def inner_func(*args, **kwargs):
        """
        This is the inner function that the timer decorator returns.
        """
        start = perf_counter()
        result = fn(*args, **kwargs)
        end = perf_counter()
        print(f'Run time: {(end - start):0.8f}')
        
        return result

    return inner_func

### Let's Unpack What's Going on Here

#### Outer Function  `timer()`

First off, `timer()` doesn't just take in the function `add()`. It can take in any function, which is why I used the generic variable `fn`. 

Note that when we put `fn` into the `timer()` function, we leave off the parentheses even though it's a function. Why did we do that? Because the parentheses tell Python to call the function, and we don't want to call it yet. If we called the function, then its output would be the input to `timer()`. Thus `timer(add(1, 3))` would actually be `timer(4)`, which is completely useless. We don't want to time the number `4`; we want to time the function `add()`. Trying to call the function without parameters, e.g., `timer(add())`, doesn't work either. You have to leave off the parentheses. 

#### Inner Function `inner_func`

`inner_func` is a generic name I picked for the function we want to time. We can call this anything we want, but it's best to pick something descriptive. 

`inner_func` is where we'll put in the parameters for the function we want to time. I could hard-code my earlier parameters `(1, 3)`, but that's not particularly useful if I want to use any other inputs.

So how do we put generic parameters in? We use `*args` and `**kwargs`.

#### What are `*args` and `**kwargs`?

These are the generic placeholder names for an arbitrary number of parameters that can be attached to any function we choose to input into `timer()`. Using one star (`*`) tells Python that this is a positional argument. Two stars (`**`) tell Python it's a keyword argument. I won't go into what positional and keyword arguments are, as parameters are a whole other conversation, but it's enough to know that `*args` and `**kwargs` serve as a catch-all for any function that we choose to time, no matter how many or what kind of parameters belong to that function.

#### Return Statements

There's no need to return the run time, since we're simply printing it. What we need to return is our original function, which I renamed `result` in the inner function. After that, we'll return `inner_func` to the outside world. This two-step process is required when writing a wrapper/decorator.

### How to Use `timer()`

Since we wrote this as a wrapper/decorator, it's not going to give the right output with a regular function call.

    timer(add(1,3))

<pre><b>[Out]</b> &lt;function __main__.timer.&lt;locals&gt;.inner_func(*args, **kwargs)&gt;</pre>

### It's Time to Decorate It

There are two ways to call a decorator. One way is like this:

    add = timer(add)

    add(1, 3)

<pre><b>[Out]</b> Run time: 0.00000065

4</pre>

We've re-assigned the name `add` to the enclosing function `timer(add)`. Now when we call `add()`, we get not just the added result, but also the run time.

Python has a prettier way to do the exact same thing. We can use the `@` sign with the decorator function's name after it. 

Make sure you don't leave a space between `@` and the function name, and don't use any parentheses. Put this on the line directly above the definition of the function you wish to decorate. 

    @timer
    def add(a, b):
        """
        Takes two parameters, a and b, and returns their sum.
        """
        return a + b
        
<pre>add(1, 3)</pre>

<pre><b>[Out]</b> Run time: 0.00000063

4</pre>

It works!

### Let's try this with another function

    @timer
    def add_more(a, b, c, d):
        """
        Takes four parameters, a, b, c, and d, and returns their sum.
        """
        return a + b + c + d

<pre>add_more(1, 3, 4, 6)</pre>

<pre><b>[Out]</b> Run time: 0.00000081

14</pre>

Woo-hoo! 

### One Tiny Problem, Though

`@timer` modified the original functions because it performed the same action as if we had re-assigned their names like so:

    add = timer(add)
    add_more = timer(add_more)
    
Re-assigning the function name means that we no longer have access to the original function's metadata. This matters if we want, for example, to get help with it.

<pre>
help(add)</pre>
    
<pre><b>[Out]</b> Help on function inner_func in module __main__:

inner_func(*args, **kwargs)
    This is the inner function that the timer decorator returns.

</pre>

<pre>help(add_more)</pre>

<pre><b>[Out]</b> Help on function inner_func in module __main__:

inner_func(*args, **kwargs)
    This is the inner function that the timer decorator returns.</pre>

Argh. Calling `help()` now gives us the doc string for `timer()`'s inner function, and this is not helpful. 

### Now What?

Python's standard library includes `functools`, which has a special tool just for the purpose of holding onto the metadata of a decorated function. It's called `wraps`, and is itself a wrapper/decorator. We can use it inside the definition of our original decorator `timer()` by decorating `inner_func`.

    def timer(fn):
    """
    This is a decorator that returns the time it takes another function to run.
    """
    from time import perf_counter
    from functools import wraps

    @wraps(fn)
    def inner_func(*args, **kwargs):
        """
        This is the inner function that the timer decorator returns.
        """
        start = perf_counter()
        result = fn(*args, **kwargs)
        end = perf_counter()
        print(f'Run time: {(end - start):0.8f}')
        
        return result

    return inner_func

Note that `wraps()` takes `fn` as a parameter the same way `timer()` does. Don't leave that out or it won't work.

Now we'll need to re-define the `add()` and `add_more()` functions so that their names point to this updated timer.

    @timer
    def add(a, b):
        """
        Takes two parameters, a and b, and returns their sum.
        """
        return a + b

    @timer
    def add_more(a, b, c, d):
        """
        Takes four parameters, a, b, c, and d, and returns their sum.
        """
        return a + b + c + d

They work just as they did before:

    add(1, 3)

<pre></b>[Out]</b> Run time: 0.00000078

4</pre>

    add_more(1, 3, 4, 6)

<pre>
<b>[Out]</b> Run time: 0.00000070

14
</pre>

But now when we call `help()`, we get what we wanted.

<pre>help(add)</pre>

<pre>
<b>[Out]</b> Help on function add in module __main__:

add(a, b)
    Takes two parameters, a and b, and returns their sum.
</pre>
        
<pre>help(add_more)</pre>
<pre>
<b>[Out]</b> Help on function add_more in module __main__:

add_more(a, b, c, d)
    Takes four parameters, a, b, c, and d, and returns their sum. </pre>

Yay!!! 

Many thanks to Dr. Fred Baptiste, whose ["Deep Dive into Python"](https://www.udemy.com/course/python-3-deep-dive-part-1/) has helped me get a better understanding of the inner workings of this language.
