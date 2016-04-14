# **Ruby** Questions


## . Main advantages and disadvantages of Ruby's dynamic typed system ?

### Advantages
+ Ruby's main advantage is that is very fast to code, also is incredible flexible, which translates on **less code**
+ Has a great community behind it, which means many gems.
+ The most popular web framework is written in Ruby. *Rails*

### Disadvantages
+ Is slower than a compile languages like: Java, C, Swift.
+ Devs could not be 100% sure if the arguments sent are empty.







## . What is the difference of *var*, *@@var*, *@var*, *$var*, *VAR* ?

### Normal Variables

The "normal variables" or just variables use on almost in any Ruby code.

```ruby
class MyClass
  def my_method(a)
    var = a
  end
end
```

### Class Variables

Class variables are share between its descendants and modules which this type of variables are define

```ruby
class MyClass
  @@var = 0
  def my_method(a)
    var = a
  end
  def my_count_method
    @@var += 1
  end
end
```

### Instance Variables

These Variables are share on an instantiate object and are initialize on the `initialize` method

```ruby
class MyClass
  attr_accessor  :var
  def initialize (var)
    @var = var
  end

  def get_my_instance_variable()
    @var
  end
end

## runtime

my_instance = MyClass.new("I am an instance variable")

puts my_instance.get_my_instance_variable()
=> I am an instance variable
```

### Global Variables:

They exist global variable on Ruby but they are not use often because messages prefer to be encapsulated between objects.

```ruby
$global_variable = "I am a global variable"
class MyClass
  # Do stuff
  def print_global_variable
    puts $global_variable
  end
end

## runtime

MyClass.print_global_variable()
=> I am a global variable
puts $global_variable
=> I am a global variable
```

### Constant Variables:

The constants are define with capital letters and within classes or modules
```ruby
VAR = "123"
class MyClass
  # Do stuff
  def print_constant_variable
    puts "the constant is: #{VAR}"
  end
end

## runtime

MyClass.print_constant_variable()
=> the constant is: 123

```







## . How to handle exceptions on Ruby ?
On Ruby different as most languages the `try-catch` block is not define, in contrast the reserved words are `begin-rescue-always`. The nex snipet will show an example:

```ruby
def my_method
  begin
    raise 'my error'
  rescue => error
    puts "#{error.class} and #{error.message}"
  end
end
```
The previous code shows how to handle errors but as a good practice is better to separate errors and not to rescue all of them, because could cause unexpected problems.

On the question [why is it bad style to rescue exception e in ruby](http://stackoverflow.com/questions/10048173/why-is-it-bad-style-to-rescue-exception-e-in-ruby) is explained much better.









## . Could you tell the difference between `super` and `super()`?

The difference between these methods is that **super**  instantiate its parent method with the child's arguments, if father and child classes have different arguments an error will raise.

On the other hand, **super( )** instantiate its parent method without any arguments. On this case, it is better to write explicit code.






## . what does it mean * `var_1 ||= var_2` * ?

Imagine if you need to assign the variable depending on something else:

```ruby
#... something else
var_2 = "some value"
if !var_1
   var_1 = var_2
end
# will return "some value" if var_1 is falsy
```
On the code above shows that `var_1` if its `falsy`. But Ruby gives a better way to write it:

```ruby
#... something else
var_2 = "some value"
var_1 ||= var_2
# will return "some value" if var_1 is falsy
```





## . On a Ruby class what does it mean *self.* before a method name means ?
On the Ruby programming language the word *self* on a class refers to the class it self. For instance, on the following snipet the difference is shown:

```ruby
class MyClass
  attr_accessor  :var
  def initialize (var)
    @var = var
  end

  def self.class_method(var)
    puts var
  end

  def instance_method
    puts "now I can have access to @var: #{@var}"
  end
end

# runtime
instance = MyClass.new("instance varible")
instance.instance_method
=> NoMethodError: undefined method 'class_method' for #<MyClass:0x007ffe5506c470 @var="instance varible"

MyClass.class_method("will call the 'self.class_method' ")
=> will call the 'self.class_method'

```
The main difference is that an class method could no be access by the instances of the class and vice versa.






## . What is a Namespace?
Namespace are the way classes or even methods separate themselves from the rest of the applications. A visible examples are the gems, they separate the classes inside module to use name spaces, by doing this they separate functionalities and prevent other users using their library accidentally open their classes and modify functionalities.

On next snipet will show how namespacing encapsulates:

```ruby
module Foo
  class MyClass
    @@baz = "fooo"
    def self.print_me
      puts @@baz
    end
  end
end

module Bar
  class MyClass
    @@baz = "barr"
    def self.print_me
      puts @@baz
    end
  end
end

## runtime

Bar::MyClass.print_me
=> barr

Foo::MyClass.print_me
=> fooo
```





## . What is the difference between *Blocks*,*Lamdas*,*Procs* ?

### Blocks
Are better known as methods







## . Given the following snipet which are true and why ?

```ruby

false  ? "true" : "false"

nil    ? "true" : "false"

true   ? "true" : "false"

1      ? "true" : "false"

0      ? "true" : "false"

"false"? "true" : "false"

""     ? "true" : "false"

[]     ? "true" : "false"  
```




```ruby

false  ? "true" : "false"
=> "false"


nil    ? "true" : "false"
=> "false"


true   ? "true" : "false"
3 => "true"


1      ? "true" : "false"
=> "true"


0 : "false"
=> "true"


"false"? "true" : "false"
=> "true"


""     ? "true" : "false"
=> "true"


[]     ? "true" : "false"
=> "true"

```





## 10. How to define methods on run time ?
