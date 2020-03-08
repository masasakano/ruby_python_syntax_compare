% Ruby vs Python

Structures
==========

## Command-line ##

| Ruby | Python  | Comment |
| :--- | :---    | :---    |
| `irb` | `python` | Interactive (`rails c` for Rails) |
| `ruby a.rb` | `python a.py` | Read script file |
| `ruby -e CODE` | `python -c CODE` | Directly gives a code |
| `-r Library` | `-m Module` | (R) Library filename; (P) Module name |
| `-d` (`--debug`) | `-d` | Debugging mode |
|  | `-u` | STDOUT/STDERR unbuffered |
| `-W[0-2]` | `-W[idmoea]` |  Verbosity level |
| `-cw a.rb` | `-m py_compile a.py` | Check syntax ((P) leaving `*.pyc`) |
| `-[anpiF]` | --- | Awk/Perl-like switches |
| --- | `-E` | (P) Ignore all environmental vars |
| --- | `-I` | (Python-3.4) Isolated mode (`-E -s` too) |

### Environmental variables and global variables/constants ###

#### Environmental variables ####

| Ruby | Python  | Comment |
| :--- | :---    | :---    |
| `RUBYLIB` | `PYTHONPATH` | Library search path |
| `RUBYOPT` | --- | Command-line option |
| `$LOAD_PATH` (`$:`) | `sys.path` | to access PATH internally |
| `$LOAD_PATH.push "mydir"` | `sys.path.append("mydir")` |  to set PATH internally |
| --- | `PYTHONDEBUG` | (P) Same as `-d` |
| --- | `PYTHONWARNINGS` | (P) [Same](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONWARNINGS) as `-W[idmoea]` |
| `ENV[:HOME]` | `os.environ['HOME']` | General access |

#### Global variables / constants ####

| Ruby | Python  | Comment |
| :--- | :---    | :---    |
| `$DEBUG` | `__debug__` | (Not sure if they mean similar...) |
| `$VERBOSE` | --- |  |
| `$LOADED_FEATURES` (`$"`)  | -- | Loaded filenames |
| `$PROGRAM_NAME` (`$0`)  | -- | Top-level script filename |
| `$FILENAME` | --- | Files being read (with `[npai]` switches) |
| `Kernel#__dir__` |  | absolute directory of `__FILE__` |
| `__FILE__` |  | (relative) path of the current source file |
|   | `__file__` | absolute path of the current source file |
| `__LINE__` |  | current line number |
| `Kernel#__method__` | `super().__getattribute__(`<br>`  cast(types.FrameType, inspect.currentframe()).f_code.co_name).name` | Current method name; (P) `import inspect, types`<br>`from typing import cast` |
| `Kernel#__callee__` |  | Current unaliased method name |
| `$stdout` | `sys.stdout` | Current Stdout |
| `$stderr` | `sys.stderr` | Current Stderr |
| `$stdin`  | `sys.stdin`  | Current Stdin  |
| `self` | --- | (P) No reserved word  |
| `nil` | `None` |   |
| `true` | `True` | ((R) `TRUE` is deprecated)   |
| `false` | `False` | ((R) `FALSE` is deprecated)  |
| --- | `NotImplemented` |   |
| --- | `Ellipsis` | (P) useful in numpy  |



## require/import ##

### Ruby ###

```ruby
  require 'ab/cd/ef'  # $RUBYLIB/ab/cd/ef.rb
  load    'ab/cd/ef'  # Reading again, forcefully
  MyNewClass.inspect  # eg., MyNewClass defined in $RUBYLIB/ab/cd/ef.rb
  include MyNewModule # Mixin
```

### Python ###

```python
  import ab.cd  # Execute ab/__init__.py and ab/cd/__init__.py
  from .ab.cd.ef import MyNewClass  # in ./ab/cd/ef.py
```

In Python, 

* the preceding `.` and `...` mean `./` and `../../` in the File-Directory structure, respectively, in the context of the Python file in which the statement appears.
* `.` in between means `/`
* The default python file to read is  `__init__.py` in the directory if the last component is a directory
* Else, the root filename is the last component

## Basics ##

### Literals and encoding ###

| Ruby | Python  | Return | Comment |
| :--- | :---    | :---   | :---    |
| `"\n"`<br>`%Q(\n)` | `"\n"`<br>`'\n'` | `"\n"` (LF) | No difference in the quotes (Python) |
| `'\n'`<br>`%q(\n)` | `r"\n"`<br>`r'\n'` | `"\\n"` (`"\u005c"+?n`) | No difference in the quotes (Python) |
| `"a" "b"` | `"a" "b"` | `"ab"` | Concatenation (you may put LF in between) |
|   | `"""Long str"""` | `"Long str"` |  |
| `"Unicode €"` | `u"Unicode €"` | | (Python 2) |
| `"\u20ac"` | `"\u20ac"`<br>`'\u20ac'` | `"€"`|  |
| `"\u20ac".ord` | `ord("\u20ac")` | `8364` |  |
| `8364.chr Encoding::UTF_8"` | `8364.chr` | `"€"`|  |
| `"\xD1xyz".b`<br>`"\xD1xyz".force_encoding('ASCII-8BIT')` | `b"\xD1xyz"` |  |  |
| `"\xD1xyz".b.encode 'UTF-8'` | `b"\xD1xyz".decode('utf-8', 'strict')` | Error | UndefinedConversionError (R), UnicodeDecodeError (P) |
| `"3 = #{1+2+a}"` | `f"3 = {1+2+a}"` | `"3 = 3.0"` | Interpolation | (where `a==0.0`) |

| Ruby | Python  | Return | Comment |
| :--- | :---    | :---   | :---    |
| `nil`  | `None`   |  |  |
| `true`  | `True`   |  | `True` is discouraged (Ruby) |
| `false`  | `False`   |  | `False` is discouraged (Ruby) |

### Regexp ###

| Ruby | Python  | Return | Comment |
| :--- | :---    | :---   | :---    |
| `/\d\n/im`<br>`%r@\d\n@im`  | `re.compile(r'\d\n',flags=re.IGNORECASE | re.M)` |  |  |

### Method (function) calls ###

In Ruby, default values in method calls are evaluated at the time of a call in the context of the method.

In Python, default values in method calls are evaluated **at the load time** of the module. Therefore, it is a bad practice to specify a mutable object (like a list and dictionary) as a default value!! In short, avoid anything but None, bool, int, float, and str!!

Also, that means you may not be able to use a constant inherited from a mixin or parent class as a default value.

An example to demonstrate the point:

```python
import time
def func(a=[], b=time.time()):  # <= b is the compile time!
    print(a);
    print(b);
    a.append('modified!')

print(time.time())
        # => 1111111111.0000001
func()  # => []
        # => 1111111110.9999999
func()  # => ['modified!']
        # => 1111111110.9999999
func()  # => ['modified!', 'modified!']
        # => 1111111110.9999999
        
def func2(c=CONST_INHERITED['VAR']): pass  # → NameError
```

#### Arguments for Method (function) calls ####

**Terminology**:  

In this subsection, an argument passed to a method/function with or without a keyword is called a positional or keyword argument, respectively.  As an analogy, in a UNIX Shell's example,

```bash
 head -n 5 a.txt b.txt
```

`-n 5` is a keyword argument with the key of `-n` (or `n`, depending on the definition), whereas `a.txt` and `b.txt` are the first and second positional arguments.

##### Ruby #####

```ruby
 def a_method(a, b=3, *c, d:, e: 8, **kwd); puts "a=#{a+1}"; end
 
 a_method('1st', '2nd', '3rd', arbitrary: 'optn' d: 'opt1')
   ## In the context of a_method():
   # a == '1st'; b == '2nd', c == ['3rd'], d == 'opt1', e == 8,
   # kwd == { arbitrary: 'optn' }
```

Ruby treats positional and keyword arguments, just like UNIX Shells. The keys of the keyword arguments should be Symbol (in Ruby 2.7 at least, I think; other types (forms) may be still allowed, but certainly not recommended!). Note Ruby 1.x does not accept keyword arguments but Hash-es.

Positional arguments must be specified in the right order.

Keyword arguments are expressed like `my_key: 123`. In the caller's context it specifies the key as a Symbol `my_key:` to be 123. In the receiver's context it means the default value of the keyword argument that is passed to the local variable `my_key` is 123 if not specified by a caller.

* 1 argument **without a keyword** (first positional argument) is mandatory (passed as an Object `a`).
* 2nd argument without a keyword (second positional argument) is optional with the default value of 3 (passed as an Object `b`).
* 3 or more positional arguments are allowed: (passed as an Array `c`)
* 1 argument with a keyword of Symbol `:d` is mandatory: (passed as an Object `d`).
* Any number of other keyword arguments are accepted (with the keys of Symbol): (passed as a Hash `kwd`)
  * The order of the keys in `kwd` is preserved

Note: A caller call this `a_method` like `a_method(xy=9, d: 5)`. In that case, `xy=9` is evaluated first, i.e., the value (an Integer instance) of 9 is substituted to the local variable `xy` in the caller's context and the statement returns the object (i.e, 9). Note the local variable `xy` in the caller's context changes (or is initialised, if not defined yet.) into 9. Second, the returned value (i.e., 9) is passed to the method `a_method` as an argument without a keyword, namely the (first) positional argument). Third, the method `a_method` receives it as the first positional argument, and hence substitutes it to its local variable `a`.

In other words, in the caller's context, any statement but one immediately followed by a single colon are just a normal statement.  In the receiver's context (in the sentence `def`), `b=9` is executed first (so the object `9` is substituted to the local variable `b`) before a value is resubstituted to `b` if a value is passed from a caller.

```ruby
 a_method(9,    d: 5)  # prints "10\n" to STDOUT
 a_method(3**2  d: 5)  # prints "10\n" to STDOUT
 a_method(57.chr.to_i,d: 5) # prints "10\n" to STDOUT  # n.b., ASCII code of "9" is 57.
 xy = 'desk'
 a_method(xy=9, d: 5)  # prints "10\n" to STDOUT
 puts xy               # prints "9\n" to STDOUT b/c xy is 9 now.
 a_method(ki=9, d: 5)  # prints "10\n" to STDOUT
 puts ki               # prints "9\n" to STDOUT b/c xy is 9 now.
```

##### Python #####

```python
 def a_method(a, b=3, *c, **kwd): print(f'a={a+1}')
  
 a_method(  1,   2,   3, arbitrary='optn')
   ## In the context of a_method():
   # a == 1; b == 2, c == [3], kwd == { arbitrary: 'optn' }
 
 a_method(  1,   2, x=3, arbitrary='optn')
 a_method(  1, b=2, x=3, arbitrary='optn')
 a_method(a=1, b=2, x=3, arbitrary='optn')
 a_method(arbitrary='optn', b=2, a=1, x=3 )
   ## In the context of a_method():
   # a == 1; b == '2nd', c == [], kwd == { x: '3rd', arbitrary: 'optn' }
 
 ## The following raise  SyntaxError
 a_method(  1, a=2)      # a to be substituted twice
 a_method(  1,   2, b=3) # b to be substituted twice
 a_method(a=1,   2)      # positional arguments must precede keyword args
 a_method(  1, b=2,   3) # positional arguments must precede keyword args
   # → SyntaxError
```

In Python, any argument is treated as both a positional **AND** keyword argument (*kind of* except for the optional arbitrary number of arguments)

In the example above, the argument that is to be substituted to the local variable `a` in the receiver's context can be specified as the first positional argument (without a keyword) or as the keyword argument with the key `a`.

The most basic rule is, 

1. any keyword argument must appear after all the positional parameters.
2. duplicated arguments (specified both as a positional and a keyword arguments) are not allowed.

The order of the keys of keyword arguments in the argument list does not matter and can be determined by the caller.

With the example above, Python handles it as follows:

* 1 argument is mandatory, which can be specified either as the 1st positional argument or as the keyword argument with a key `a` at the first location (passed as an object `a`).
* 2nd argument is optional with the default value of 3, which can be specified either as the 2nd positional argument or as the keyword argument with a key `b` that can appear anywhere after all the other positional arguments (passed as an object `b`).
* 3 or more positional arguments without a keyword is allowed, as long as the keyword arguments with the key of `a` or `b` are not given: (passed as an list `c`)
* Any number and key of other keyword arguments are accepted and in an arbitrary order, as long as they all appear after all the positional arguments: (passed as a dictionary `kwd`)
  * The order of the keys in `kwd` is preserved (in Python 3.6+)

### Return ###

In Ruby, any statement returns an object.

In Python, substitution (including those in place like `+=`) statements return nothing; it would be *SyntaxError* to try to refer to it:

```python
  print(a+=[9])  # => SyntaxError
```

### True/False ###

| Ruby | Python  | Return | Comment |
| :--- | :---    | :---   | :---    |
| `if 0`  | `0==False`  | True |  |
| `if 1`  | `1==True`   | True |  |
| `if 2`  | `2!=True`   | True |  |
| `if ""` | `""!=True`  | True |  |
| `if ""` | `""!=False` | True |  |
| `if ""` | `if not ""` | True |  |
| `if (2 and "2")` | `(2 and "2")` | True |  |

In Ruby, anything but `false` (`False`) and `nil` is treated as true.

In Python, it is messy…  
I **think** Python's rule is

1. "*Not Truthy*" includes `None`, `False`, (int) `0`, and anything that appears empty, including (str) `""`, (list) `[]`, (dict) `{}`, (set) `set()`, thought not limited with these.
2. Whether "*Truthy*" or not is judged in the `if` condition and other conditional statements, such as, `xy` in `(xy and True)`
3. Usually, when two objects are **equal** to each other, where "**equal**" means `a == b` returns `True`, their values and their types must match more or less (though it depends on each class' definition of its `__eq__()` method; e.g., `1 == 1.0` returns `True`).
4. "*False*" and "*True*" are the boolean instances, and they are not **equal** to anything else other than themselves with one exception for each.
   1. **Exception**: Numeric `0` (i.e., int `0` and float `0.0`) is equal to `False` (`(0 == False)` returns `True`)
   2. **Exception**: Numeric `1` (i.e., int `1` and float `1.0`) is equal to `True`  (`(1 == True)`  returns `True`)
   3. `('1' == True)`  returns `False`
   4. `(''  == True)`  returns `False`
   5. `(''  == False)` returns `False`
   6. `('0' == False)` returns `False`
   7. `(None == False)` returns `False`
   8. `0` is falsy. `''` is falsy. However, `(0 != '')` because they are in different types.

(Personally, this specification of regarding only some random Numeric to be equal to a Boolean value is awful…  A Numeric instance may by chance end up in `0.0` after some complex calculation, possibly just because of inherent limitation in floating-point calculation, and then it suddenly becomes equal to `False`.  How come!?  And how random!)

### Conventions ###

In Ruby, a method with a suffix of `!` is more destructive than its counterpart without. What it returns depends.

In Python, when a method is destructive, a counterpart may be defined as a function, e.g., `ary.sort()` vs `sorted(ary)`.  A destructive method should in principle return `None` (and hence [Fluent Interface](https://en.wikipedia.org/wiki/Fluent_interface) (aka method cascading) cannot be used). Note there are exceptions like `pop()` and `popitem()` for `dict`.

## Control structure ##

### Loop ###

| Ruby | Python  | Comment |
| :--- | :---     | :---    |
| `loop do; end` | `while True:` | Indefinite loop |
| `next` | `continue` | (P) `next()` is a very **different** function!<br>In Emacs, `next` is in *blue*.  |
| `redo` | --- |   |
| `break` | `break` |  |
| --- | `else` | executed **only if** not break-ed  |
| `retry` | --- | (R) retry from a `rescue` clause.  |
| `begin`<br>`rescue IOError => e` `#`(may `retry`)<br>`else`<br>`ensure`<br>`end` | `try:`<br>`except(IOError) as e:`<br>`else:`<br>`finally:` | To catch an exception  |
| `abc rescue def` | --- | (R) one liner to catch `StandardError` |
| `raise IOError, 'msg'`| `raise IOError(msg)` |   |
| `catch(:myid){ throw :myid, obj}`| --- | (P) No escape from a nested loop  |

#### Ruby example

```ruby
 loop {
   begin
     next if false
     redo if true
     break
   rescue Errno::SyntaxError =>x
     retry
     warn 'warn-danger'  # Taking account of -W, -d
     raise NameError, 'HiThere'
   else
   ensure
   end
 }
  
 # break from a nested loop
 catch(:escape){loop{loop{throw :escape,123}}}
```

For `StandardError`, a shortcut is

```ruby
 a.no_method rescue a.other_method
```

#### Python example

```python
 import warnings
 import logging
 logging.basicConfig(level=logging.DEBUG) # filename='mylog.log'
   # DEBUG, INFO, WARNING (Default), ERROR, CRITICAL
 while True:
     try:
         x = int(raw_input("Please enter a number: "))
         if False: continue
         break
     except IOError, (errno, strerror) as e:
         print "I/O error(%s): %s" % (errno, strerror)
         print "msg=", str(e)	# Bare "e" is accepted in Python2, but not in 3.
     except (ValueError, TypeError), inst:  # "as" is mandatory in Python 3?
         print type(inst), inst.args
     except:  # Omission of Error class name is generally discouraged.
         warnings.warn('warn-danger', category=None)
         logging.warning('Watch out!')  # more debug-output
         logging.debug('Debug...')      # very verbose output
         raise NameError('HiThere')
     else:  ## if no error is raised.
         pass
     finally:  ## ALWAYS executed!  NOTE: it cannot coexist with "except"!!
         pass
```

No built-in control structure to break out of a nested loop.
See [a neat way](https://stackoverflow.com/questions/189645/how-to-break-out-of-multiple-loops/3171971#3171971).

### Iterators ###

| Ruby | Python  | Comment |
| :--- | :---     | :---    |
| `ar.each do |i|; end` | `for i in ar:` | Array |
| `ar.each_with_index do |i, v|; end` | `for i, v in enumerate(ar):` | Array with index |
| `ar1.zip(ar2).each do |i|; end` | `for i,j in zip(ar1, ar2):` | Two arrays |
| `hs.each_pair do |k, v|; end` | `for i, v hs.items():` | Hash with key |
| `hs.each_key do |ek|; end` | `for ek in hs.keys():`<br>`for ek in hs:` | Hash keys |
| `hs.each_value do |v|; end` | `for v hs.values():` | Hash values |
| `ar.map{|i| i-1}` | `[i-1 for i in ar]` | Python list comprehension |
| `ar.keep_if{|i| i<2}.map{|i| i-1}`<br>`ar.filter_map{(_1<2) ? _1 : nil}` | `[i-1 for i in ar if i < 2]` | (P) conditional list comprehension<br>(R) 2nd form for Ruby-2.7+ |
| `hs.map{|k, v| [k, v.upcase]}.to_h` | `{a: b.upper() for a, b in hs.items()}` | (P) dict comprehension |
|   | `(i-1 for i in ar if i < 2)` | Python list generator |


## Conditional structure ##

| Ruby | Python  | Comment |
| :--- | :---     | :---    |
| `a=5 if b==2` | `if b==2: a=5` |  |
| `a=5 if b==2` | `a=5 if b==2 else (a or None)` | `else`-clause is mandatory;<br>*NameError* if `a` is undefined in Python |
| `a=(b==2 ? 5 : 6)` | `a=5  if b==2 else 6`<br>`a=(5 if b==2 else 6)` | inline condition; (P) `if`-clause is **for 5** |
| `a ||= 5` | `if a is None or a is False: a=5` | substitute if nil or false (Ruby) |
| `a = 5 if a==0 ||`<br> `  a.respond_to?(:empty?) && a.empty?` | `a=(a or 5)` | substitute if false-**ish** (0, `""`, `[]` **etc**) (**Python!!**) |
| `a = 5 if a.nil?` | `if a is not None: a=5` | substitute if a is null |
| `case a;when Integer, 'x';else;end` | --- | (P) No case/switch structures |

## Operators ##

| Ruby | Python  | Comment |
| :--- | :---     | :---    |
| `a=5 if b==2` | `if b==2: a=5` | inline condition |
| `a=5 unless b==2` | `if not b==2: a=5` | negative condition |
| `a_if ? b_then : c_else` | `b_then if a_if else c_else` | Ternary |
| `m = (nil or 'Significant')`<br>`m = nil || 'Significant'` | `m = None or 'Significant'` |  |
| `(m = nil) || (n = 5)` | --- | |
| `!!some` | bool(some) | converts to Boolean |
| `2 <= a && a < 5`<br>`(2...5).cover? a`<br>`(2...5) === a` | `2 <= a < 5` | (R) `2 <= a < 5` raises `NoMethodError` |

In Python, substitution statements return nothing; hence the third last form does not exist.

In Ruby, every operator is a method except for the *isolated* equal sign `=` and two (or four) logical operators (`&&` and `||`, plus `and` and `or`). This means mathematical-looking operators such as `*` and `<` and `<=` and `==` are methods, which can be written as `.*`, `.<`, `.<=`, and `==`, and accordingly can be redifined by the user.  Therefore, `2 <= a < 5` is interpreted as `1.<=(a).<(5)`; the first part yields `true` and hence it reduces to `true.<(5)`. The `TrueClass`, which `true` belongs to, does not have the method `<`, and hence `NoMethodError` is raised.

Class
====================

| Ruby | Python  | Comment |
| :--- | :---     | :---    |
| `class MyC` | `class MyC` | (P) must never reappear!! |
| `class MyC < Parent` | `class MyC(Parent, Grandparent)` | inheritance |
| `class < inst` | --- | Singleton class |
| `include Mix1` | `class MyC(Mix1, Parent)` | mixin declaration |
| `def inst_method(x); self;` | `def inst_method(myself, x): myself` | Instance method |
| `def self.class.cl_method(x)` | `@classmethod`<br>`def cl_method(cls, x):` | Class method (see below) |
| `private; def stat_method(x)` | `@staticmethod`<br>`def stat_method(x):` | Static method (see below) |
| `initialize()` | `__init__(self)` |  |
| `attr_reader :myat`<br>`def initialize;@myat=9;end`<br>`MyC.new.myat` | `def __init__(self):`<br>`  self.myat=9`<br>`MyC().myat` |  |
| `attr_writer :myat`<br>`a=MyC.new; a.myat=8` | <br>`a=MyC(); a.myat=8` |  |
| `MyC.new.amethod` | `MyC().amethod()`<br>`m=MyC().amethod; m()` |  |
| `MyC.new.class` | `type(MyC())`<br>`MyC().__class__` | <br>(Python 2/3) |
| `MyC.ancestors` | `MyC.__mro__` | (P) Method Resolution Order |
| `MyC.descendants` | --- | All descendants |
| `MyC.descendants[0]` | `MyC.__subclasses__` | (R) Single inheritence |
| `MyC.included_modules` | `MyC.__bases__` | nb., including the Parent class! (Python) |
| `MyC.new(8)` | `MyC(8)` | Creates an instance |
| `(a.instance_variables + a.methods).sort` | `dir(a)` | Get all *attributes*. |
| `a.instance_variables.sort` | `list(filter(lambda i: not callable(getattr(a, i)), dir(a)))` | Get instance variables |
| `a.instance_variables.sort` | `list(filter(lambda i:     callable(getattr(a, i)), dir(a)))` | Get methods |
| `a.instance_variable_get(:@var)` | `a.var`<br>`getattr(a, 'var', DEF)` | Get an instance variable (or method). (R) Highly discouraged.<br> (R) maybe nil (P) may raise AttributeError |
| `a.instance_variable_set(:@var, 8)` | `a.var=8`<br>`setattr(a, 'var', 8)` | Set an instance variable (or define method). (R) Highly discouraged.<br>(R, P) never raises an Exception |
| --- | `property(f_get,f_set,f_del,f_doc)` | (P) Set up the filter for getter/setter |
| `a.method(:metho)` | `a.metho` | (P) Get a function instance. |
| `a.metho 1, 'x'`<br>`a.method(:metho).call(1, 'x')` | `a.metho(1, 'x')`<br>`b=a.metho; b(1, 'x')` | Instance method call.<br>(P) Parentheses trigger the call. |
| `MyC.metho 1, 'x'` | `MyC.metho(1, 'x')` | Class method call. |
| `MyC::MY_CONST` | --- | Class constant |


| Ruby | Python  | Comment |
| :--- | :---     | :---    |
| `class MyC; def new_m` | `def new_m(): pass`<br>`setattr(MyC, 'new_m', new_m)` | Add a new method. (P) Discouraged. |
|  | `class MyC: from myfile import new_m` | (P) To import in the **sole** class definition. |

Or in Python, it is better to define your own child class inherited from the parent class, where you define your own additional methods.  Then create instances of the child class and use them.  Obviously this strategy would not work well for the built-in classes such as String and List, as you cannot redefine the literals such as `[]` (for List). To be fair, it is debatable whether it is an acceptable practice to modify the behaviour of such basic classes.

| Ruby | Python  | Comment |
| :--- | :---     | :---    |
| `to_s` | `__str__(self)` |  |
| `inspect` | `__repr__(self)` |  |
| `+(val)` | `__add__(self, val)` |  |
| --- | `__iadd__(self, val)` | `a+=b` equivalent to `a=a+b` (Ruby, Python default) |
| `>(val)` | `__gt__(self, val)` |  |
| `[](key1, *key2)` | `__getitem__(self, key)` | `key` is passed as Tuple (when more than 1?) (Python) |
| `[]=(key1, key2, *key3)` | `__setitem__(self, key, val)` | `key` is passed as Tuple (when more than 1?) (Python) |

In Python, the following is allowed:

```python
class MyD: pass
class MyC(str):
    def __init__(self, s):
        super().__init__()
    def get_upper(self):  # Instant method
        return self.upper()

 ## Expected
MyC("xyz")  # => "xyz"
MyC("xyz").get_upper()  # => "XYZ"
MyC("xyz").get_upper    # => <bound method MyC.get_upper of 'xyz'>

 ## Try to use it as a class method(!)
MyC.get_upper          # => <function MyC.get_upper at 0x104677c10>
MyC.get_upper()        # TypeError: get_upper() missing 1 required positional argument: 'self'
MyC.get_upper("xyz")   # => 'XYZ'
```

In Ruby, to refer to and set an instance variable should be always made via a getter/setter method explicitly defined for that purpose in the class of the instance.  Three module functions are available for convenience to define them like `Module#attr_writer(:var)`. If a class has, for example, `attr_writer :var`, then the instance variable `var` of its instance `a` is set as `a.var=8` (because `Module#attr_writer(:var)` is simply a short hand of `def var=(x); @var=x; end`).

## Class/instance methods ##

In Ruby, the class method is a singleton method of a Class instance.  In other words, class methods and instance methods are independent in Ruby. Private methods in Ruby cannot be called with a receiver, hence cannot be accessed from outside of the class definition (except with some special methods); but self and class can be accessed from them, unlike static methods in Python.

In Python, any method can be called from the class that define them, hence is technically a class method as well as an instance method.  However, if an instance method is called directly from a class, the first argument is **not** passed automatically(!!) and it must be given **explicitly**. Confusing? Yes, never do it!!

With the `@classmethod` decorator, you can specify a method to be the class method. They are especially handy for alternative constructors. The class method **can** be called from an instance, and in that case a **class is passed** as the first argument, as opposed to **self**, just like a class calling a class method. Confusing? Yes, don't do it.

Static methods in Python offers nothing more than a namespace. Note they can be accessed from outside of the class definition and you cannot hide them. In Python, everything is public after all. In short, there is the following relation (kind of):

    Static method ⊂ Class method ⊂ Instance method

In Ruby, the class method is a singleton method of a Class instance.  In other words, class methods and instance methods are independent of each other in Ruby. Private methods in Ruby cannot be called with a receiver, hence cannot be accessed from outside of the class definition (except with some special methods); but self and class can be accessed from them, unlike static methods in Python.

## Inheritance and mixin ##

Ruby's model is single inheritance with mix-ins. By contrast, Python's model is multiple inheritance. Therefore, in Python, the list of the *ancestors* is non-trivial, such the famous (notorious?) [Diamond Problem](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem) ([菱形継承問題](https://ja.wikipedia.org/wiki/%E8%8F%B1%E5%BD%A2%E7%B6%99%E6%89%BF%E5%95%8F%E9%A1%8C)) and Python has (must have) its own **Method Resolution Order (MRO)**  (`MyC.__mro__`; nb., *variable* as opposed to a method) (see "[The Python 2.3 Method Resolution Order](https://www.python.org/download/releases/2.3/mro/)" for the algorithm, which is the defunct definitive guide applicable for Python 2.3+, including Python 3). In practice in Python, it is recommended to *mimic* a single inheritence with added parent classes of "*virtual*" mix-ins; they do not redefine their own initialisation but only define some methods common to their descendants and are *included* by its direct child classes as semi-parent, whereas the class that is the child's meant-to-be parent is treated like the grand-parent or more senior ancestor, i.e., 

```python
  class MyC(MixinParent, MixinGrandParent, BaseParent):
```

In Ruby, the structure of class A that has mixin M and A's child B that has mixin M is allowed. For the class B, the method defined in mixin M has a precedence over that in A. In Python, this structure is not allowed as it violates its MRO rule; in Python mixin M is technically an ancestor (parent) of class A, for it does not have the designated concept of the mixin.


References:

* "[The Python 2.3 Method Resolution Order](https://www.python.org/download/releases/2.3/mro/)" (aka MRO)
* [Ian Lewis's article](https://www.ianlewis.org/en/mixins-and-python)
* [Marcus Lind's article](https://coderbook.com/@marcus/deep-dive-into-python-mixins-and-multiple-inheritance/).

## Class variable ##

In Ruby, 

* An instance variable of a class `@vari` is literally an instance of a particular class, which in itself is an object of a `Class` class.
  * it is completely independent of an instance variable of an instance.
  * it is not accessible from a scope of an instance of the class.
* A *class variable* `@@varv` is a class variable shared over **all descendant** classes ([Stackoverflow](https://stackoverflow.com/questions/15773552/ruby-class-instance-variable-vs-class-variable/15773671#15773671)). (Hence it may cause an unexpected side effect.)
  * Again, completely independent of an instance variable of an instance.
  * it **is** accessible from a scope of an instance of the class, in addition to a scope of the class.

Example:

```ruby
class MyC
  @@varv = 'I am a class var'
   @vari = 'I am a class instance'
  def self.show_sh; @@varv; end
  def self.show_it; @vari; end
  def not_show_it; @vari; end
end
class Chi < MyC
end
MyC.show_sh          # => "I am a class var"
MyC.show_it          # => "I am a class instance"
MyC.new.not_show_it  # => nil
Chi.show_sh          # => "I am a class var"
Chi.show_it          # => nil
Chi.new.not_show_it  # => nil
```

In Python, a class variable of a class is **shared** with all its instances (and is accessible from the outside world, as everything is public in Python).  For an instance it is **shared, too**, although it can separate once an object ID changes (which is true if the class variable is immutable).

A quote from [Official Reference](https://docs.python.org/3/tutorial/classes.html#class-and-instance-variables):

> shared data can have possibly surprising effects with involving mutable objects such as lists and dictionaries

Example:

```python
class MyC:
    vari = 'I am a class instance'
    varN = { 'a': "Extra careful!" }

class Chi(MyC): pass

m__ = MyC()
MyC.vari           # => 'I am a class instance'
m__.vari           # => 'I am a class instance'
MyC.vari = 'change by class'
MyC.vari           # => 'change by class'
m__.vari           # => 'change by class'
m__.vari = 'change by instance'
MyC.vari           # => 'change by class'
m__.vari           # => 'change by instance'  (Now it is independent.)
MyC().vari         # => 'change by class'  (New instance inherits from a class.)
Chi.vari           # => 'change by class'
Chi().vari         # => 'change by class'

MyC.varN           # => {'a': 'Extra careful!'}
MyC().varN         # => {'a': 'Extra careful!'}
MyC().varN['a'] = 'Unexpected change.'
MyC().varN         # => {'a': 'Unexpected change.'}
MyC.varN           # => {'a': 'Unexpected change.'}
```

Built-in functions (Python)
===========================

| Ruby | Python   | Return | Comment |
| :--- | :---     | :---   | :---    |
| `Enumerable#all?` | `all(iter)` | bool | True if empty? or all of them are true |
| `Enumerable#any?` | `any(iter)` | bool | True if any of them is true |
| `Object#remove_instance_variable(:val)`<br>`Module#undef_method(:met)` | `delattr(obj, 'attr')`<br>`del obj.attr` |  | Delete/remove an attribute<br>(R) distinguishes an instance var & method  |
| `Object#methods.sort` | `dir(obj)` | Array/list | instances are not a method (Ruby)<br>Explicitly aimed for debugging use (Python) |
| `Object#instance_variables` | `obj.__dict__` | Array/list |   |
| `Object#send(NAME, args)` | `getattr(obj, NAME)(args)` | Object | Dynamically call a method |
| `Object#instance_variable_defined?(NAME)` | `hasattr(obj, NAME)` | Boolean | |
| `Object#instance_variable_defined?(NAME) ? \\`<br>`Object#instance_variable_get(NAME) : DEF` | `getattr(obj, NAME, DEF)` | Object | Dynamically get an attr.<br>(Ruby) Do **NOT** do it!|
| `Object#instance_variable_set(NAME, VAL)` | `setattr(obj, NAME, VAL)` | | Dynamically set an attr.<br>(Ruby) Do **NOT** do it!|
| `Object#id` | `id(obj)` | Integer |  |
| `Object#is_a? klass`<br>`Object#kind_of? klass` | `isinstance(obj, klass)` | Boolean |  |
| `open(fname, 'r')` | `open(fname, mode='r')` | IO/file | `buffering=n` [can be specified](https://docs.python.org/3/library/functions.html#open) (Python) |
| `open(fname, 'r'){|io| }` | `with open(fname, mode='r') as io:` |  |  |
| `Object#to_s` | `str(obj)` | String |  |
| `Object#inspect` | `repr(obj)` | String | Double(Ruby) and Single(Python) quotes for String |
| `Object#class` | `type(obj)`<br>`obj.__class__` | Class |  |
| `super` | `super()` |  |  |


Standard data classes
====================

## String ##

| Ruby | Python  | Comment |
| :--- | :---     | :---    |
| Normal instance | **Immutable** instance |  |
| `String#capitalize` | `str#capitalize()` |  |
| `String#swapcase`   | `str#swapcase()`   |  |
| `String#downcase`   | `str#lower()` |  |
| `String#upcase`     | `str#upper()` |  |
|                  | `str#title()` | Title-case: "it's" → "It'S" |
| `String#split(nil|String, 5)` | `str#split(None|String, 5)` |  |
| `String#split(nil|String, 0)` | `str#split(None|String, None)` |  |
| `String#split(nil|String, -1)` | --- |  |
| `String#split(/abc/, n)` | --- |  |
| `String#strip` | `str#strip()` |  |
|  --- | `str#strip('aab')` | Duplication of chars is irrelevant |
| `String#rstrip` | `str#rstrip()` |  |
| `String#sub(/[\n\r]*$/, '')` | `str#rstrip('\n\r')` | Any combination of `[\n\r]` (Python) |
| `String#chomp` | --- | 1 character (Ruby). |
| `String#[n]` | `s[n]` | (Ruby 2) |
| `String#[n]=` | --- | *TypeError* (Python) |


**NOTE**: Because String in Python is immutable, they have no *destructive* methods.

### print ###

| Ruby | Python  | Comment |
| :--- | :---     | :---    |
| `puts a` | `print(a)` |
| `print a," "` | `print(a),` (Python 2)<br>`print(a, sep='', end=" ")` (Python 3) |
| `print a` | `sys.stdout.write(a)` (Python 2)<br>`print(a, end="")` (Python 3) | with `import sys` |
| `$stderr.puts a` | `print(a,file=sys.stderr)` (Python 3) | |
| `puts a; $stdout.sync` | `print(a,flush=True)` |  |

### sprintf ###

| Ruby | Python  | Comment |
| :--- | :---     | :---    |
| `"Var is #{5+3}"` | `f'Var is #{5+3}'` |  (Python 3.6+) |
| `"it is %s and %d "%(v1, v2)` | `"it is %d and %s "%(v1, v2)` | (*old style* in Python) |
| `sprintf("it is %s and %d", v1, v2)` | --- | |
| --- | `'No, {} at {:9.4f}'.format('Bob', 2.1)`| | 
| --- | `'{1} {0} {2:4.2f}'.format('two', 'one', 3.14159))` | (→ `"one two 3.14"`) |
| --- | `'{name} has 0x{num:x}'.format(name='Bob', num=31)` | (→ "3f") | 

## Array/List ##

| Ruby | Python  | Comment |
| :--- | :---     | :---    |
| `[][0] == nil` | ``[][0] #=> IndexError` |  |
| `[true]*3`<br> `Array.new(3, true)` | `[True]*3` | for immutables  |
| `Array.new(3){?_}` | `['_']*3` | for non-immutables  |
| `Array.new(3){Array.new(3){?_}}` | `[['_']*3 for i in range(3)]` | **NOT** `[['_']*3]*3` |
| `Array#size`,<br> `Array#length` | `len(ary)` |
| `Array#map{|i| i-3}`<br>`Array#map{_1-3}` | `[i-3 for i in ary]` | Python list comprehension<br>(R) 2nd form for Ruby-2.7+ |
| `Array#map{&:upcase}` | `[i.upper() for i in ary]` |  |
| `Array#keep_if{|i| i<2}.map{|i| i-1}`<br>`Array#filter_map{(_1<2) ? _1 : nil}` | `[i-1 for i in ary] if i < 2` | (P) conditional list comprehension<br>(R) 2nd form for Ruby-2.7+ |
| `Array#reverse` | `reversed(ary)` |  |
| `Array#reverse!` | `ary.reverse()  |` |
| `Array#sort!` | `ary.sort()` |  |
| `Array#sort!{|a,b| a.size<=>b.size}` | `ary.sort(key=len)` |  |
| `Array#sort!{|a,b| a.upcase<=>b.upcase}` | `ary.sort(key=str.upper)` |  |
| `Array#sort!{|a,b| a<=>b}` | `ary.sort(key=lambda a: a)` | `a` must be comparable in Python |  |
| `Array#sort!{|a,b| (a<=>b) || 0 }` | --- | If incomparable sometimes; (P) No direct ways |
| `ary.sort.reverse` | `sorted(ary, reverse=True)` |
| `ary.grep(/^__/)` | `list(filter(lambda i: re.search(r'^__', i), ary))` |

Suppose `a=a1=[0,1,2,3]` and `b=a2=['x','y']`

| Ruby | Python  | Original | Return | Comment |
| :--- | :---    | :---     | :---   | :---    |
| `a1 + a2` | `a1 + a2` |  | `[0,1,2,3,'x','y']` |
| `a1 += a2` | ---  | | `[0,1,2,3,'x','y']` | Object-ID changes |
| ---        | `a1 += a2` | `[0,1,2,3,'x','y']` | *NOT defined* | `__iadd__()` |
| `a[1]`     | `a[1]` |  | 1 | referring |
| `a[0..]` | `a[:]` |  | [0,1,2,3] | `0..-1` before Ruby 2.6 |
| `a[1..2]`  | `a[1:3]`  |  | [1,2]  | referring |
| `a[1..-2]` | `a[1:-1]` |  | [1,2]  | referring |
| `a[1..-1]` | `a[1:]`   |  | [1,2,3]  | referring |
| `a[-3, 2]`  | --- |  | [1,2] | referring by size |
| `a[5]=9`   |          | `[0,1,2,3,nil,9]` | 9 | index creation & assignment |
|            | `a[4]=9` |  | IndexError | (index out of range) |
| `a[1]=[5,6,7]`    | `a[1]=[5,6,7]` | `[0,[5,6,7],2,3]` |  | single-element substitute |
| `a[1..2]=[5,6,7]` | `a[1:3]=[5,6,7]`<br>`a[1:3]=(5,6,7)` | [0,5,6,7,3] | `[5,6,7]` (Ruby) | substitute by range |
| `a[1..2]=*('XY'.split(''))` | `a[1:3]='XY'` | `[0,"X","Y",3]` |  | **Treated as iterable** in Python |
| `a[1,2]=[5,6,7]` | --- | [0,5,6,7,3] | `[5,6,7]` | substitute by size |
| --- | `a[::3]` | | [0,3] | Specify a step |
| `a[1,0]=9` | `a[1:0]=[9]` | [0,9,1,2,3] | 9 (Ruby) | insert; **Must be iterable** in Python |
| `a.insert(1, a2)` | `a.insert(1, a2)` | `[0,['x','y'],1,2,3]` | new `a` (R) or None (P) | insert a single object |
| `a.insert(1, 5, 6)` | --- | `[0,5,6,1,2,3]` | `[0,5,6,1,2,3]` |  |

### Tuple (Python) ###

Ordered and unchangeable collections (or containers).

**WARNING**: Do **NOT** put mutable items in tuples!!  See an example:

```python
t = (1, 2, [30, 40])
try:
    t[2] += [50, 60]  # => raises TypeError
except TypeError:
    pass

print(t)  # (1, 2, [30,40,50,60]) (despite TypeError!!)
```

### NArray ###

`NArray` in Ruby is `array.array` in Python (example from (Fluent Python (p.52))):

```python
from array import array
from random import random
floats = array( 'd', (random() for i in range(10**7)))
```

or `numpy`

```python
import numpy
a = numpy.arange(12)
a.shape = 3, 4  # adding a dimension.
```

## Hash/Dictionary (Map Abstract Data Type (ADT)) ##

Key differences are summarised in [Stackoverflow](https://stackoverflow.com/questions/23575603/what-is-the-difference-between-a-ruby-hash-and-a-python-dictionary/23575714#23575714).
Notably,

1. When a missing-key is looked up with `[]`,
   * Hash(Ruby): the key is created with a Default value (`nil`, unless pre-defined)
   * dict(Python): `KeyError`
     * `dict.setdefault(key, None)` (providing `nil` is a Default in the Ruby Hash) is equivalent to Ruby's `h[key]`.
   * [collections.defaultdict](https://docs.python.org/3/library/collections.html#collections.defaultdict)(Python): The same as Hash(Ruby), except when `None` (or nothing) is specified as the Default (in which case it is the same as built-in `dict`).
     * **Note**: `h.get()` still returns `None` regardless of the Default if the key is missing.
     * **Note**: To set the default as `None`,
       ```python
         from collections import defaultdict
         defaultdict(lambda: None)
         defaultdict(lambda: None, {'a': 8})
       ```
2. Ability to use built-in mutable types (eg. lists) as keys: Hash(Ruby) only
3. Insertion-ordering guarantees: yes (Ruby2 and Python3.6 or later only)
   * cf. `collections.OrderedDict` in Python



| Ruby | Python  | Return | Comment |
| :--- | :---    | :---   | :---    |
| ---  | `dict(one=1, two=2)` | | Constructor |
| `{'one'=>1, 'two'=>2}` | `{'one': 1, 'two': 2}` |  | Constructor |
| `Hash[{'one'=>1, 'two'=>2}]` | `dict({'one': 1, 'two': 2})` |  | Constructor |
| `{one: 1, two: 2}`<br>`{:one=>1, :two=>2}` | --- |  | Constructor with Symbol keys |
| `Hash[['one', 'two'].zip [1, 2]]` | `dict(zip(['one', 'two'], [1, 2]))` |  | Constructor |
| `Hash[[['one', 1], ['two', 2]]]` | `dict([('one', 1), ('two', 2)])` |  | Constructor |
| `Hash[[[1, 'x'],[2,'y']].map{|k, v|[k, v.upcase]}]` | `{k: v.upper() for k, v in [(1, 'x'),(2,'y')]}` |  | dict comprehension |
| `Hash#size`   | `len(h)` | Integer |  |
| `Hash#empty?` | `if (len(h)==0):` | Boolean |  |
| `Hash#keys`   | `h.keys()` | Array/[view](https://docs.python.org/3/library/stdtypes.html#dictionary-view-objects) | It used to be List (in Python 2) |
|               | `list(h)` | list |  |
| `Hash#values` | `h.values()` | Array/[view](https://docs.python.org/3/library/stdtypes.html#dictionary-view-objects) | |
| `Hash#to_a`   | `h.items()` | Array[Array]/[view](https://docs.python.org/3/library/stdtypes.html#dictionary-view-objects)[Tuple] | |
| `Hash#has_key? k`<br>`Hash#key? k`<br>`Hash#include? k`<br>`Hash#member? k` | `k in h` | Boolean | `has_key` in Python 2.2 |
| `! Hash#has_key?(k)` | `k not in h` | Boolean |  |
| `Hash#delete k` | `del(h[k])` | `k` or nil/None or [KeyError](https://docs.python.org/3/library/exceptions.html#KeyError) | nil if not found (Ruby) |
| `Hash#delete k` | `h.pop(k, DEFAULT)` | /k or DEFAULT | [KeyError](https://docs.python.org/3/library/exceptions.html#KeyError) if DEFAULT is unspecified and k is missing (Python) |
| --- | `reversed(h)` | dict | From Python 3.8 |

(Dictionary) [view](https://docs.python.org/3/library/stdtypes.html#dictionary-view-objects) object is a dynamic window (or symlink) of dict keys etc, as opposed to a copy at the time of referring.

Suppose `h=h1={'a'=>1,'b'=>3}` and `k=h2={'c'=>'y'}`

| Ruby | Python  | Original | Return | Comment |
| :--- | :---    | :---     | :---   | :---    |
| `h.clear` | `h.clear()` | `{}` |  |  |
| `h1.merge! h2` | `h1.update(h2)` | `{'a'=>1,'b'=>3,'c'=>'y'}` | | |
|  | `h1.update({5: 99})` | `{'a'=>1,'b'=>3,5=>99}` | | |
|  | `h1.update(g=99})` | `{'a'=>1,'b'=>3,'g'=>99}` | | |
| `h1.merge h2` | `dict(h1, **h2)` | `h1` | `{'a'=>1,'b'=>3,'c'=>'y'}` | |

| Ruby | Python  | Comment |
| :--- | :---     | :---    |
| `h[nonexistent]` | `h.setdefault(nonexistent, DEFAULT)` | `{'a'=>1,'b'=>3,nonexistent=>DEFAULT}` | `nil`/DEFAULT | or Default set by `h.default=` etc (Ruby). |
| `h.fetch(nonexistent)` | `h[nonexistent]` | `h` | KeyError | |
| `h.fetch(nonexistent, DEFAULT)` | `h.get(nonexistent, DEFAULT)` | `h` | DEFAULT |  |
| `h.fetch(nonexistent){|k| }` | --- | `h` | Return of Block |  |
| `h.each_key{|ek| }`    | `for ek in h:`<br>`for ek in list(h):` |  | | Iterator over keys |
|                        | `for ek in h.keys():` |  | | Dynamic (Python 3). Equivalent(?) to `.iterkeys()` in Python 2 |
| `h.each_pair{|k, h| }` | `for k, h in h.items():` |  | | Iterator over pairs |
| `h.each_value{|ev| }`  | `for ev in h.values():` |  | | Iterator over values |
| `h.filter{ |k, v| v<=2}` | `{k: v for k, v in h.items() if v <= 2}` | | Hash | |
| `h.filter!{|k, v| v<=2}`<br>`h.select{ |k, v| v<=2}` | `for k,v in list(h.items()): if v>2: del(h[k])` | `{'a'=>1}` | Hash or nil | (Ruby) returns nil if no changes made. |
| `h.keep_if{|k, v| v<=2}` |  | `{'a'=>1}` | Hash | (Ruby) returns always self |

## IO-related classes ##

| Ruby | Python  | Original | Return | Comment |
| :--- | :---    | :---     | :---   | :---    |
| `IO#close` | `io.close` |  | |  |
| `IO#closed?` | `io.closed()` |  | Boolean |  |
| `IO#flush` | `io.flush()` |  | |  |
| `IO#read [size]` | `io.read([size])` |  | | `nil` (Ruby) or negative Integer (Python) means up to EOF |
| `IO#gets [maxsize]` | `io.readline([maxsize])` |  | | At most maxsize characters if specified. `nil` at EOF (Ruby) |
| `while line=IO#gets {}` | `for line in f: print(line)` |  | | |
| `IO#readline [maxsize]` | --- |  | `EOFError` at EOF | |
| `IO#readlines [maxsize]` | `IO#readlines [maxsize]` |  | Array/list | |
| `IO#sync` | [`io.line_buffering`](https://docs.python.org/3/library/io.html#io.TextIOWrapper.line_buffering) |  |  | In Text-mode only (Python) |
| `IO#sync=BOOL` | `open(buffering=[0,1,-1])` |  |  | 0(unbuffering/Binary), 1(line/Text), -1(system/Def) (Python) |
| `IO#write s` | `io.write(s)` |  | | Returns Integer (character number) (Python) |
| `IO#write(s1, s2, s3)` | `io.writelines(ary)` |  | | Linefeeds are **not** added. (Python) |
| `IO#print s` | `io.write(s)` |  | | |
| `IO#puts s` |  |  | | a newline is guaranteed (Ruby) |

### Shell interaction ###

In Python, make sure `import subprocess` first.  `PIPE`, `STDOUT` etc are its attributes.
Here assuming, `run`, `check_output` too are imported.

In Ruby, `require open3` is the standard for the extra functionality.

| Ruby | Python  | Original | Return | Comment |
| :--- | :---    | :---     | :---   | :---    |
| `%x(echo 'hi')` | `check_output("echo 'hi'", shell=True, encoding='utf-8')` |  |  |
| \``cat naiyo 2>&1`\` | `check_output('cat naiyo', shell=True, encoding='utf-8',`<br>` stderr=STDOUT)` |  |  |
| `system('cat naiyo')`<br>`$?` | `r=run(['cat', 'naiyo'], encoding='utf-8')`<br>` r.returncode` |  |  |
| `o,e,s = Open3.capture3("echo a; sort",`<br>` :stdin_data=>"f\nb\n")` | `r=run("echo a; sort", shell=True, encoding='utf-8',`<br>` input="f\nb\n", capture_output=true)`<br>`(r.stdout,r.stderr,r.returncode)` |  |  |
| `rescue Errno::EXXX` | `except (TimeoutExpired, FileNotFoundError)` |  |  |

### Common ways ###

#### Ruby ####

```ruby
open(fname, 'r'){|io| 
  while line=io.gets do
    line.chomp!
    puts line
  end
}

open(fname, 'w'){|io| 
  io.sync=true
  10.times do
    io.puts 'Something'
  end
}
```

#### Python ####

```python
with open(fname, 'r') as io:
    for line in io:
        line.rstrip('\n\r')  # any combination of \n and \n
        print(line)
}

with open(fname, 'w', buffering=1) as io:  # sync=true
    for i in range(10):
        io.write(line.rstrip() + '\n')
}
```

## Regexp ##

| Ruby | Python  | Return | Comment |
| :--- | :---    | :---   | :---    |
| `Regexp.escape('ab*d')`<br>`Regexp.quote('ab*d')` | `re.escape('ab*d')` |  |  |
| `/\A5.*\nab\Z/m` | `re.compile(r'^5.*\nab$', flags=re.DOTALL)` |   | (R,P) Multiline means opposite!! |
| `/^5.*\nab$/`    | `re.compile(r'^5.*\nab$', flags=re.M)` | |  |
| `m = %r@E?MS([1-2])@.match s`<br>`/E?MS([1-2])/ =~ s` | `m = re.search(r'^E?MS([1-2])', s, flags=re.IGNORECASE)` | `MatchData` | (R) `$1, $&` etc available |
| `[m[0], m[1]]` | `[m.group(), m.group(1)]` |  |  |
| `str.gsub(/[a-f]/){|x| x.upcase}` | `re.sub(r'[a-f]', lambda x: x.upper, str)` |  | |
| `str.sub(/(a)(c)/, '\2\1')` | `re.sub(r'(a)(c)', '\2\1', str, 1)` |  | |

## Duck-typing ##

| Ruby | Python  | Return | Comment |
| :--- | :---    | :---   | :---    |
| `Object#to_s` | `str(obj)` | String |  |
| `Object#inspect` | `repr(obj)` | String | Double(Ruby) and Single(Python) quotes for String |
| [`Object#respond_to?`](http://ruby-doc.org/core/Object.html#method-i-respond_to-3F) `metho` | `if callable(getattr(obj, "metho", None)):`<br>`if hasattr(obj, metho) and callable(obj.__getattribute__(metho)):` | Boolean | To check a method |

## Miscellaneous ##

| Ruby | Python  | Return | Comment |
| :--- | :---    | :---   | :---    |
| `File.basename __FILE__`<br>`File.basename $0` | `os.path.basename(__file__)` | str  | (P) `import os` |

