# Failing test case minimizer

This is a POC implementation of the _delta debugging_ algorithm for generating
1-minimal failing test cases.  The algorithm is described in:

_Simplifying Failure-Inducing Input_, Ralf Hildebrandt and Andreas Zeller, 2000.

[PDF download](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.126.6907&rep=rep1&type=pdf).

# Demo

Consider `bad.py`:

```python
#!/usr/bin/env python2

print 'hello there, how are you'

import sys
import os

os.system('ls')

sys.exit(42)
```

Running this code will result in a non-zero exit code:

```
$ python bad.py
hello there, how are you
bad.py  ddmin
$ echo $?
42
```

Let's try to minimize this test case:

```
$ ddmin -i bad.py -o bad0.py "python @input"
...
$ cat bad0.py
#
```

That was not very helpful.  The problem is that `ddmin` regards any non-zero
exit code as a failure, and running the "program" consisting of a single `#`
will make python fail.  So let's only regard an exit code of 42 as an error:

```
$ ddmin -i bad.py -o bad1.py --status 42 "python @input"
...
$ cat bad1.py
import sys
sys.exit(42)
```

Much better.  We can also consider the presence of the string `"hello there"` in
the output as an error and get this minimal test case:

```
$ ddmin -i bad.py -o bad2.py --writes "hello there" "python @input"
...
$ cat bad1.py
print'hello there'
```
