#Data Structures  

[Arrays](http://www.ruby-doc.org/core-2.1.1/Array.html)  |  [Hashes](http://www.ruby-doc.org/core-2.1.1/Hash.html)  
## Arrays  
In Ruby, we define an aray simply using square brackets []  
```ruby
a = [1, 59, "Hey", "Yo", 200]
a[2]                  # "Hey!!"
```  
You can put anything inside an array  
```ruby
a = "hey"
my_first_array = [1, "Hello", true, false, nil, a]
my_first_array[3]     # false
my_first_array.first  # 1
my_first_array.last   # "hey"

```  

You can have an array inside an array  
```ruby
my_array = [1, "Hello", [1, 2, 3], "Hey", nil]
my_array[2]           # [1, 2, 3]
my_array[2][1]        # 2
```  
__Multi-Dimensional Arrays__
`[[1,2,3],[4,5,6],[7,8,9]]`  
To push elements into an array in ruby, we use `<<`   or `push`  
```ruby
my_array = [1,2,3]
my_array << 4         # [1, 2, 3, 4]
my_array.include? 5   # false
```  
1.) Find out how to get the number of Array elements in two different ways.  
2.) Find out how to turn a multi-dimensional Array into a one dimensional Array  
```ruby
# 1.)
my_array = [1,2,3,4,5,6]
my_array.length
my_array.count
my_array.size

3 2.)
my_array = [1,2,3,[4,5,6,[7,8,9],10],11,12]
my_array.flatten
```  
3.) Find out how to convert a string sentence to an array of words  
```ruby
str = 'This is a bunch of words'
str.split(' ')
```  
4.) FizzBuzz: Write code that adds 1 to 100 to an array and if code is divisible by 3, then adds FIZZ, if it's divisible by 5, adds Buzz, and if it's divisible by both put FIZZBUZZ.  
```ruby
#FizzBuzz
1.upto(100) do |n|
  if n % 15 == 0
    puts "FIZZBUZZ"
  elsif n % 5 == 0
    puts "BUZZ"
  elsif n % 3 == 0
    puts "FIZZ"
  else
    puts n
  end
end
```  
## Iterrating through an Array  
```ruby
my_array = [1, 5, 6, 7]
my_array.each do |x|
```  
5.) Build an array that contains five names, then loop through the array and print names capitalized.  
```ruby
names = ["joe", "ellen", "marissa", "martin", "franki"]

names.each do |n|
  puts n.capitalize
end
```  
6.) Write code that prints every element in the two-dimensional array. **Stretch**: Do the same thing in a totally different way  
```ruby
arr = [[1,2,3],[1,2,3],[1,2,3],[1,2,3]]
puts arr
puts arr.flatten

arr.each {|x| puts x}

arr.each do |outer|
  outer.each do |inner|
    puts inner
  end
end
```  
7.)
```ruby
my_array.map {|x| x * x}

#is equivalent to
my_array.map do |x|
  x * x
end

names = ['frank', 'ed', 'jo', 'kristy', 'elise']

# returns the original array
names.each do |name|
  name.capitalize
end

# returns an array of capitalized elements
names.map do |name|
  name.capitalize
end
```  
## Hashes   
```ruby
me = {"name" => "Tam"}

puts me["name"]
```  
Pass a default value  
```ruby
personal_info = Hash.new("Unknown")

personal_info["name"]         # "Unknown"

occurances = Hash.new(0)
occurances["s"]               # 0
```  

1.) Write a hash that stores your personal information  
```ruby
my_info = Hash.new('Not Submitted')
my_info = {
  "name" => "Drew",
  "city" => "Vancouver",
  "favorite food" => "Pasta",
  "favorite sport" => "Bacci Ball"
}
```  
2.) Find a method that will return an array of all the keys in your hash  
```ruby
my_info.keys
```  
3.) Write a hash that contains three Canadian provinces as keys and their capital as values, then loop through it and print each province and its capital  
```ruby
provinces = {"British Columbia" => "Victoria", "Ontaria" => "Toronto", "Nunavut" => "Iqualuit"}
provinces.each_pair {|k, v| puts "The capital of #{k} is #{v}."}
```  
Just as we can have hashes inside arrays, we can have arrays as values inside hashes  
```ruby
my_hash = {"abc" => [1,2,3]}
my_array = ["abc", {"a" => 1, "b" => 2}, "def"]
```  
4.) Build a hash that contains three Canadian provinces as keys and the values are arrays that contain three cities each.  
```ruby
provinces = {
  "BC" => ["Vancouver", "North Vancouver", "West Vancouver"]
  "AB" => ["Calgary", "Edmonton", "Red Deer"]
  "WA" => ["Seattle", "Bellingham", "Blaine"]
}
```  
## Symbols    
We define a symbol by placeing a colon before it. Symbols are *immutable* - this means that once you have created a symbol, its value cannot be altered. Accessing a symbol is generally faster than accessing a string (they get saved in a more excellent spot in memory).  
```ruby
a = :bc
b = :bc
a.object_id     # returns the object id
b.object_id     # returns the same object id as a

a = "bc"
b = "bc"
a.object_id     # returns the object id
b.object_id     # returns a different object id from a

# If you define your hash with symbols, you access its values with symbols
my_hash = {:a => 1, :b => 2, :c => 3}
my_hash[:a]
```  
You can convert symbols to strings and strings to symbols using `to_s` and `to_sym`, respectively.  
```ruby
:bc.to_s        # "bc"
"bc".to_sym     # :bc
```  
5.) Write a hash that stores your personal information using symbols  
```ruby
my_info = Hash.new('Not Submitted')
my_info = {
  :name => "Drew",
  :city => "Vancouver",
  :favorite_food => "Pasta",
  :favorite_sport => "Bacci Ball"
}
```  
## Methods   
When we wish to perform an operation, or set of operations multiple times, we can write a method  
```ruby
# Write a method that takes two arguments, and returns their product
def multiply(a, b)
  a * b
end
```  
6.) Write a method that takes two arguments, and returns their sum  
```ruby
def sum(a, b)
  a + b
end
```  
In order to make arguments optional, you can give them default values.  
```ruby
def multiply(a, b=1)
  a * b
end
```  
7.) Write a method that takes two arguments and print the results of first to the power of the second.  
```ruby
def power_it(a, b)
  a ** b
end
```  
What will the following methods return, if a = 2 ,and b = 3?  
```ruby
def my_method(a, b)
  a * b
  a + b                       #returns a + b
end

def my_method(a, b)
  return a * b                #return a * b
  a + b
end
```  
8.) Write a method called by_five? that takes a single number parameter, and returns true f that number is evenly divisible by five and false if not. Don't use return in this case.  
```ruby
def by_five?(a)
  if a % 5 == 0
    return true
  else
    return false
  end
end

# By default ruby will return the last line, so we do not need to call 'return true'
def by_five?(a)
  if a % 5 == 0
    true
  else
    false
  end
end

# This can be further refactored
def by_five?(a)
  a % 5 == 0
end
```  
9.) Write a method that takes variable number of numbers and returns the sum of all these numbers  
```ruby
sum = (1, 2, 3, 4, 5, 6, 7, 88)
def sum(a, *b)
  result = a
  b.each {|x| result += x}
  result
end

def sum(*a)
  sum = 0
  a.each {|x| sum += x}
  sum
end

def sum(*args)
  args.inject(:+)
end
```  
## More Data Structures  
__Recursion__

```ruby
my_array = ["Hello world", true, [1, 2, 3], [4, 5, 6, [7, 8, 9], 10], 11]

def find_arrays_inside(array)

  array.each do |element|
    if element.is_a? Array
      puts "I encountered an array with size #{element_size}"
      find_arrays_inside(element)
    end
  end
end
```  

__More fun with Hashes__   
```ruby
my_hash = {:a => "Hello"}

# This notation only works in Ruby 1.9 and above
my_hash = {a: "Hello"}
```  

## Blocks
A block is a piece of code with no name (an anonymous function)  
yield
```ruby
def my_method
  puts "Hello"
  yield if block_given?
  puts "Bye"
end

# Everything between 'do' and 'end' is the block
my_method do
  puts "I'm a block"
end
```
Using yield to display a block
```ruby
def my_method
  puts "Hello"
  yield if block_given?            # Will run the given block if one is given
  puts "Bye"
end

my_method do
  puts "Hello CodeCore"
  x = 2 + 2
  puts x
end
```  
Passing a variable in with a block  
```ruby
def method2
  puts ">>>>>>>>>>>>>>>"
  x = 100
  yield(x)
  puts ">>>>>>>>>>>>>>>"
end

method2 {|x| puts x * x}
method2 {|X| puts x + x}
```  
## Procs & Lambdas  
Lambdas are a special type of proc. [differences between procs and lambdas](https://gist.github.com/ogryzek/9224008)
```ruby
my_first_lambda = { puts "Hello CodeCore" }
my_first_lambda.call
```  

```ruby
my_lambda = lambda { puts "Im a lambda!" }
my_proc = proc { puts "I'm a proc" }

def my_method(piece_of_code)
  puts "Before"
  piece_of_code.call
  puts "After"
end

my_method(my_lambda)
my_method(my_proc)
```  
Some differences between procs and lambdas (Test in IRB)
```ruby
my_lambda = lambda { puts "Im a lambda!" }
my_proc = proc { puts "I'm a proc" }

my_lamdba do
  puts "I'm a lambda"
  return
end

my_proc do
   puts "I'm a proc"
   return
end

my_lamdba do |x|
  puts "I'm a lambda - x is #{x}"
  return
end

my_proc do |x|
  puts "I'm a proc - x is #{x}"
  return
end
```  
# Exceptions  
Exceptions happen when the user does something that shouldn't be done. For example, when we try to divide by zero, we get `ZeroDivisionError: divided by 0`. Or, if we try to call variable b, which does not exist, we get `NameError: undefined local variable or method 'b' for main:Object`. If we try to call a method on nil that doesn't exist nil.asdasd, we get `NoMethodError: undefined method 'asdasd' for nil:NilClass`.  

If we want to raise a specific error, we can by using raise. Try `raise ZeroDivisionError` in IRB.
```ruby
# Handling Exceptions
begin
  10/0
  rescue ZeroDivisionError => e
    puts "You cannot divide by zero -- #{e.message}"
end

# failing gracefully
begin
  puts "Hello"
  10/0
  rescue ZeroDivisionError => e
    puts "You cannot divide by zero -- #{e.message}"
  puts "Bye"
end

# versus failing hard
begin
  puts "Hello"
  10/0
  rescue ZeroDivisionError => e
    raise ZeroDivisionError
  puts "Bye"
end

begin
  asdfasd
  rescue => e
  puts "Something when wrong!"
end

class MyException < StandardError
end

raise MyException, "Some exception happened"
```  
# Tidbits  
__Ternary Operator__  
```Ruby
puts "Give me a number"
number = gets.chomp

if number > 10
  puts "Your number is large."
else
  puts "Your number is small"
end

number > 10 ? puts "Your number is large" : puts "Your number is small"

puts number > 10 ?"Your number is large" : "Your number is small"
```  
10.) Write FizzBuzz using a ternary operator  
```Ruby
# FizzBuzz in one line
1.upto(100) {|x| puts (x % 15 == 0) ? "FizzBuzz" : (x % 5 == 0) ? "Buzz" : (x % 3 == 0) ? 'Fizz' : x}
```  

__Using Times__  
```Ruby
10.times {|x| puts "hey guys #{x}" }

x = 15
x.times { puts "Hey" }
```  

__Conditions__
```Ruby
# 'if' and 'unless'
puts "Hello" if x < 25
puts "Hello" unless x < 25

# Conditional assignment
b ||= 10
b ||= 4

a = 15
a ||= 6

c ||= 8

b = nil
b = 10 unless defined?(b) || b.nil?
```  
__Case When__  
```Ruby
puts "What do you speak?"
language = gets.chomp

# this case/when statement is...
case language
when "English" then puts "Hello"
when "French" then puts "Bonjour"
when "Spanish" then puts "Hola"
else puts "I don't speak #{language}"
end

# ... equivalent to this if/else statement.
if language == "English
  puts "Hello"
elsif language == "French"
  puts "Bonjour"
elsif language == "Spanish"
  puts "Hola"
else
  puts "I don't speak that #{language}"
end
```  

__Operator Fun__
```ruby
def a
  puts "A was evaluated!"
  return true
end

def b
  puts "B was also evaluated!"
  return true
end

puts a || b
puts "_______"

# Test if a variable responds to a method
a = "Hello CodeCore"
b = 100

a.respond_to?(:to_i)

a.respond_to? :capitalize
b.respond_to? :capitalize
```  
11.) Build a method that takes 1 argument, then print out the string capitalized only if it can be capitalized. If not, display "Input can't be captalized"  
```ruby
def can_cap?(str)
  if str.respond_to? :capitalize
    puts str.capitalize
  else
    puts "Input can't be captalized"
  end
end
```  
__Extra Tidbits__  
```ruby
#
my_string = 100
Float(my_string) rescue false         # 100

my_string = "hello"
Float(my_string) rescue false         # false
