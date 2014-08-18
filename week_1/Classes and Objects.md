# Classes & Objects  
Before object oriented programming, there was what's called 'Procedural Programming'. If I wanted to describe the mundane takss of my day to day routine, I could do so using procedures, such as "I wake up. I brush my teeth, etc."

However, we as humans tend to think of everything as an object. This is why languages like Ruby take objects to the extreme by making everything objects. Object oriented programming is good, because it makes thing easier to understand, and easier to extend. I can just include a library and have access to many more objects and methods.

I get into a car, which is an object. I turn on the car, which is a method on the object. Object interacting with each other is what OOP is all about.

Most modern languages are object oriented. Php, Java, Scala, etc.  
  
__Some Differences between a Class and an Object__  
A cookie is an object. An easy way to create many cookies is with a cookie cutter. A cookie cutter is like a class. I should create a class once, then I can create many objects. So, we use a cookie cutter to create many cookies.  
Or you can think of a class as a blueprint. I have one blueprint, I can build many houses.  
Of course, in Ruby everything is an object, even the cookie cutter, but don't worry about that for now.  
Let's visualize this in code.  
## Code  
To create a class, I can just define it, like this:
```ruby
class Cookie

end
```  
Now, I can call `Cookie.new` to create as many cookies as I want  
```ruby
cookie1 = Cookie.new
cookie2 = Cookie.new
cookie3 = Cookie.new
```  
Play around with it in IRB a little. Create a file called cookie.rb with the class Cookie defined. To load the file in irb, you can use `load "./cookie.rb"` if you are in the same directory. Then instantiate some cookies, and check their object ids.  
```ruby
cookie = Cookie.new
cookie.object_id
cookie = Cookie.new
cookie.object_id
cookie2 = Cookie.new
cookie2.object_id
cookie.object_id
```  
By convention, classes have their first letter `capitalized`, while files and methods use `snake_case`. It is advised to name the file, the same as the class. And to have 1 class per file, or 1 file per class.  
  
__1.) Build a class for Car and a class for British Columbia.__  
(Try to put each in a different file)  
```ruby
# car.rb
class Car

end
```  
```ruby
# british_columbia.rb
class BritishColumbia

end
```  
Now that we have classes, we can add methods to them  
```ruby
# car.rb
class Cookie

  def bake
    "baking baking..."
  end
  
  def eat
    "Nom Nom Nom"
  end
end
```  
We can call these methods on objects of this class in IRB  
```ruby
load './cookie.rb'
cookie = Cookie.new
cookie.bake             # "baking baking..."
cookie.new              # "Nom Nom Nom"
```
Practice writing some methods of your own.  
  
##Public and Private Methods  
We can add methods that are publicly accessible, but sometimes we want to have private methods within our class. Generally, we add private methods at the bottom of the class file. Actions (methods) within the class, can access the private methods, but private methods can not be called directly from outside of the class.  
  
In this case, the `bake_and_eat` method has access to the private method `eat`, so you could call `cookie.bake_and_eat` from outside the class. However, you can no longer call `cookie.eat`.  
```ruby
# cookie.rb
class Cookie

  def bake
    "baking baking..."
  end
  
  def bake_and_eat
    bake
    eat
  end
  
  private 
  
  def eat
    "Nom Nom Nom"
  end
end
```  
Test it out  
```ruby
cookie = Cookie.new
cookie.bake
cookie.eat                # this will throw an error.
cookie.bake_and_eat
```  
2.) Add public methods to your car: drive, stop, and park that just print text.  
```ruby
# car.rb
class Car

  def drive
    puts "driving..."
  end
  
  def stop
    puts "stopping..."
  end
  
  def park
    puts "parking..."
  end

end
```  
3.) Add private methods to your car: pump gas, and ignite engine.  
```ruby
class Car

  def drive
    puts "driving..."
  end
  
  def stop
    puts "stopping..."
  end
  
  def park
    puts "parking..."
  end
  
  private
  
  def pump_gas
    puts "pumping gas."
  end
  
  def ignite_engine
    puts "starting the car."
  end

end
```  
4.) Call the ignite_engine method before printing on the drive method.  
```ruby
class Car

  def drive
    ignite_engine
    puts "driving..."
  end
  
  def stop
    puts "stopping..."
  end
  
  def park
    puts "parking..."
  end
  
  private
  
  def pump_gas
    puts "pumping gas."
  end
  
  def ignite_engine
    puts "starting the car."
  end

end
```  
## Class Methods  
A class method is a method that begins with the keyword `self`. If I have a method that begins with the class, rather than an instance of that object, it is a class method.  
```ruby
class Cookie
  def self.who_are_you?
    puts "I am a cookie"
  end
end

cookie = Cookie.new
cookie.who_are_you?     # will give an error
Cookie.who_are_you?
```  
Here's an example from Ruby using `Random`.  
**note**: If you have a class in ruby you don't know, you can call `methods` on it. So, `Random.methods` will list all the methods available to that class.  
```ruby
Randon.name
Random.new_seed  # This is a class method. I'm calling it directly on the class, without creating an object.
```  
They could have implemented it that I say `r = Random.new`, then `r.new_seed` to get this.  

5.) Add a class method max_speed to your Car class that returns 200.  
```ruby
class Car

  def self.max_speed
    200
  end

  def drive
    ignite_engine
    puts "driving..."
  end
  
  def stop
    puts "stopping..."
  end
  
  def park
    puts "parking..."
  end
  
  private
  
  def pump_gas
    puts "pumping gas."
  end
  
  def ignite_engine
    puts "starting the car."
  end

end
```  
Define a private class method, for example
```ruby
# car.rb
class Car
  #...
  
  def stop
    # max_speed             # will not work
    Car.max_speed           # If you need to call a class method from within a non class method, you can
    puts "stopping..."
  end
  
  def self.max_speed
    absolute_speed = 100
  end
  
  private
  
  def self.absolute_speed
    300
  end
  
  #...

end

```  

##Defining an intialize method for a class  
`initialize` is a special ruby method. Always called `initialize`. It is called when you first create an object.
```ruby
class Car

  def initialized
    puts "I am initialized"
  end

  def drive
    ignite_engine
    puts "driving..."
  end
  
  def stop
    puts "stopping..."
  end
  
  def park
    puts "parking..."
  end
  
  def stop
    puts "stopping..."
  end
  
  def self.max_speed
    absolute_speed = 100
  end
  
  private
  
  def self.absolute_speed
    300
  end
  
  def pump_gas
    puts "pumping gas."
  end
  
  def ignite_engine
    puts "starting the car."
  end

end
```  
To add instance variables add an `@` before the variable name.  
```ruby
# car.rb
class Car
  #...
  def initialize(fuel_amount, passenger_count)
    @fuel_amount = fuel_amount
    @passenger_count = passenger_count
    puts "I'm in initialize"
  end
  
  #...

  def fuel_amount
    @fuel_amount
  end
  
  #...

end  
```  
You can either give default values, or it will through an error if the correct number of arguments aren't given. Add some default values to the `initialize` method  
```ruby
# car.rb
class Car
  #...
  def initialize(fuel_amount=0, passenger_count=0)
    @fuel_amount = fuel_amount
    @passenger_count = passenger_count
    puts "I'm in initialize"
  end
  #...
end
```  
6.) In your car class, add the following attributes to the initialize: model, type and capacity
```ruby
# car.rb
class Car
  #...
  def initialize(fuel_amount=0, passenger_count=0)
    @fuel_amount = fuel_amount
    @passenger_count = passenger_count
    puts "I'm in initialize"
  end
  #...
end
```  

7.) Build a class "computer" that has parameters: brand, memory, and cpu power  
```ruby
# computer.rb
class Computer
  
  # add an initialize method

end

```  
## Global Variables  
You can create a global variable with a `$`. This will be accessible _everywhere_  
```ruby
class Cookie
  $variable = "Hello world!"
end
```  

You can also use `@@` to create a global that is accessible from every instance of an object. For example
```ruby
class Cookie
  @@color = "Brown"
end
```  
**note**: Any instance of Cookie could now access and change @@color.  

If for example, we define we can define abc in the car initialize method, and then define a method to set it  
```ruby
# car.rb
class Car
  def initialize(fuel_amount=0, passenger_count=0, model="ford", type="sedan", capacity=5)
    @fuel_amount = fuel_amount
    @passenger_count = passenger_count
    @model = model
    @type = type
    @capacity = capacity
    @@abc = 1
    puts "I'm in initialize"
  end
  
  def abc
    @@abc
  end
  
  def set_abc(a)
    @@abc = a
  end
  
  #...
end
```  
Now, we can access and set @@abc from any instance of Car, and it will change it for all instances of car. Try it in irb
```ruby
load 'car.rb'
car1 = Car.new
car2 = Car.new

car1.abc
car2.abc

car2.set_abc(75)
car1.abc
```

## Class naming convention  
By convention, classes in Ruby begin with a Capital letter. It is good to follow conventions to prevent unexpected behavior. Here's an example of overwriting a class.  
```ruby
class Computer; end
Computer.new
Computer = 10           # (irb):47: warning: already initialized constant Computer
                        #  (irb):46: warning: previous definition of Computer was here
Computer.new            # NoMethodError: undefined method `new' for 10:Fixnum
```
  
## Adding attribute writers to methods  

```ruby
class Cookie
  def sugar_amount
    @amount
  end

  def sugar_amount=(amount)         # this is called a setter in many programming languages
    @sugar_amount = amount
  end
end
```
  
Using [attr_accessor](http://ruby-doc.org/core-2.1.0/Module.html#method-i-attr_accessor), [attr_reader](http://ruby-doc.org/core-2.1.0/Module.html#method-i-attr_reader) and [attr_writer](http://ruby-doc.org/core-2.1.0/Module.html#method-i-attr_writer)
```ruby
# car.rb
class Car
  def initialize(fuel_amount=0, passenger_count=0, model="ford", type="sedan", capacity=5)
    @fuel_amount = fuel_amount
    @passenger_count = passenger_count
    @model = model
    @type = type
    @capacity = capacity
    @@abc = 1
    puts "I'm in initialize"
  end
  
  # attr_writer :passenger_count      # this is an eqivalent setter
  def passenger_count=(count)         # setter : used to set a value
    @passenger_count = count
  end
  
  # attr_reader :passenger_count      # this is an equivalent getter
  def passenger_count                 # getter : used to get a value
    @passenger
  end
  
  # attr_accessor :passenger_count    # This is both a getter and a setter
  
  def abc
    @@abc
  end
  
  def set_abc(a)
    @@abc = a
  end
  
  #...
end
```  
8.) Build a class "Rectangle" and give it two attribute accessors: width and height. Add a method "area" that returns the area 
```ruby
# rectangle.rb
class Rectangle
  attr_accessor :width, :height
  
  def initialize(width, height)
    @width, @height = width, height
  end
  
  def area
    width * height
  end

end
```  
or  

```ruby
# rectangle.rb
class Rectangle
  attr_accessor :width, :height
  
  def area
    width * height
  end

end
```  
**Note**: If you have a method variable, and you have an instance variable, it's better to access the instance variable.  
  
```ruby
# rectangle.rb
class Rectangle
  # attr_accessor :width      # we could remove this, and in this case, it will still be OK.
  attr_accessor :height
  
  def initialize(width, height)
    @width, @height = width, height
  end
  
  def area
    width * height            # now we are accessing the attribute width through the method width
  end
  
  def width
    @width * 2.2
  end

end  
```  
**Note** (again): When you have the option to access a variable through an instance variable or a local variable, *always* choose the local variable. In ruby, the way we call a method or a variable, they're the same.  

10.) Write the class that is defined in this file  
```ruby
reqire './test_attr_methods.rb"
foo = TestAttrMethods.new("hi")
foo.greetings
> "hi Hello! bonjour!"
foo.b = "Hola!"
foo.greetings
> "Hi Hola! bonjour!"
foo.c = "abc"
foo.greetings
> "Hi Hola! abc"
foo.a = "Welcome!"
> NoMethodError: undefined method 'a'...
```  
```ruby
#test_attr_methods.rb
class TestAttrMethods
  attr_accessor :b
  attr_writer   :c
  
  def initialize(a, b="Hello!", c="Bonjour!")
    @a, @b, @c = a, b, c
  end
  
  def greetings
    puts "#{@a} #{b} #{c}"
  end
  
end
```  
## Inheritance  
Classes can inherit from other classes which gives them access to their methods.
```ruby
# cookie.rb
class Cookie
  attr_accessor :sugar_amount
  attr_accessor :flour_amount
  
  def initialize(sugar_amount, flour_amount)
    @sugar_amount, @flour_amount= sugar_amount, flour_amount
  end
  
  def name
    puts "I'm a cookie"
  end

end
```  
The class Oreo inherits the class Cookie
```ruby
# oreo.rb
require 'cookie.rb'
class Oreo < Cookie

  def name 
    puts "I'm an oreo"
    super               # super calls the method with the exact same name in the parent class
  end

end
```  
```ruby
# cream.rb
require 'oreo.rb'
class Cream < Oreo

  def name
    super               # calls the method 'name' defined in class Oreo.
  end
  
end
```
Test it out in irb  
```ruby
cookie = Cookie.new(5, 4)
cookie.flour_amount = 10
cookie

oreo = Oreo.new         # will fail, because Oreo inherits initialize from Cookie
oreo = Oreo.new(6, 5)

oreo.is_a? Oreo
oreo.is_a? Cookie
cookie.is_a? Oreo
cookie.is_a? Cookie

oreo.name
```  
## Modules  
### Modules as namespaces
We can have different classes with the same names that belong to different modules. Imagine how we might organize apples.    
```ruby
module Fruit
  class Apple
  end
end

module Computer
  class Apple
  end
end

# To instantiate these classes, we use ::
apple_computer = Computer::Apple.new
apple_fruit = Fruit::Apple.new
```   
An alternative syntax  
```ruby
class Computer::Apple
end

class Fruit::Apple
end

apple_computer = Computer::Apple.new
apple_fruit = Fruit::Apple.new
```  
### Modules as Mixins  
You can give modules methods.  
```ruby
module SpecialModule
  def cool_method
    #...
  end
end

class SomeClass
  attr_accessor :name
  
  include SpecialModule               # include a model to 'mixin' its methods
  
  def do_something
    cool_method, name
  end
end





```