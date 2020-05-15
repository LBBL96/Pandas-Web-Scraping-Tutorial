# How to Write and Use a Decorator in Python
## @ isn't just for email anymore

### What's a Decorator?
In Python, a decorator is a function that wraps another function. Its purpose is to provide some additional functionality to the function that it wraps.

Let's look at an example.

Let's say you've written a simple function that adds up two numbers. You also put a doc string in there because you're awesome and want to help anyone who uses your function to understand what it does.

    `def add(a, b):
    """
    Takes two parameters, a and b, and returns their sum.
    """
    return a + b`

    `add(1, 3)
    [out] 4`

I want to see what your function does, so I call help on it. 

    `help(add)`

    `[out] Help on function add in module __main__:

    add(a, b)
    Takes two parameters, a and b, and returns their sum.
`

