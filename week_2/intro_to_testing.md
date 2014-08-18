[Codewars Tests](https://github.com/Codewars/kata-test-framework-ruby) | [RSpec](http://rspec.info/) | [Minitest](http://ruby-doc.org/stdlib-2.1.1/libdoc/minitest/rdoc/MiniTest.html) | [FizzBuzz(Custom)](http://www.codewars.com/kata/5355a811a93a501adf000ab7/train/ruby) | [Find the Capitals](http://www.codewars.com/kata/53573877d5493b4d6e00050c/train/ruby) | [Name Array Capping](http://www.codewars.com/kata/5356ad2cbb858025d800111d/train/ruby)
  
# Testing in Ruby  
Testing can be a kind of documentation for what you want your code to do. It helps not only with debugging, but it helps you organize the ideas you are trying to implement. Weeks, months, or years later it helps to see what a particular piece of code was intended to do, and if you add or modify code that breaks tests, you'll have a clearer idea of why and where.  
  
Let's build a class called Person. Objects of this class should instantiate with up to three parameters: name, age, occupation. If no parameters are given, then the default value for each of the parameters should be the string "unknown". We should be able to instantiate a new person object, then later modify these attributes with setter and getter methods.  
  
How would we translate these instructions for coding the Person class into tests?
```ruby
# codewars_person_test.rb

require './person.rb'
require './framework.rb'

describe Person do
  describe "with no parameters given" do
    before do
      @person = Person.new
    end

    describe "#name" do
      it "should return 'unknown' if no parameters given" do
        Test.expect(@person.name == "unknown")
      end

      it "should be editable" do
        Test.expect(@person.name = "Johnny" == "Johnny", "expected: 'Johnny', got #{@person.name = 'Johnny'}")
      end

    end

    describe "#age" do
      it "should return 'unknown' if no parameters are given" do
        Test.expect(@person.age == "unknown")
      end

      it "should be editable" do
        Test.expect(@person.age = 15 == 15, "expected: 15, got #{@person.age}")
      end
    end

    describe "#occupation" do
      it "should return 'unknown' if no parameters are given" do
        Test.expect(@person.occupation == "unknown", "expected: 'unkown', got #{@person.occupation}")
    end

      it "should be editable" do
        Test.expect(@person.occupation = 'doctor' == 'doctor')
      end
    end
  end

  describe "with parameters given" do
    before do
      @person = Person.new("Nelly", 26, "Teacher")
    end

    describe "#name" do
      it "should return 'Nelly'" do
        Test.assert_equals(@person.name, 'Nelly')
      end

      it "should be editable" do
        Test.assert_equals(@person.name = 'Corey', 'Corey')
      end
    end

    describe "#age" do
      it "should return the age" do
        Test.assert_equals(@person.age, 26)
      end

      it "should be editable" do
        Test.assert_equals(@person.age = 27, 27)
      end
    end

    describe "#occupation" do
      it "should return the occupation" do
        Test.assert_equals(@person.occupation, 'Teacher')
      end

      it "should be editable" do
        Test.assert_equals(@person.occupation = 'English Teacher', 'English Teacher')
      end
    end
  end
end
``` 
And what would person look like?  
```ruby
# person.rb

class Person
  attr_accessor :name, :age, :occupation
  def initialize(name="unknown", age="unknown", occupation="unknown")
    @name, @age, @occupation = name, age, occupation
  end
end
```
Now, let's try replicating these tests with Minitest and RSpec
```ruby
# minitest_person_test.rb

require 'minitest/autorun'
require './person.rb'

class TestPerson < Minitest::Test

  describe Person do
    describe "when no parameters are given" do
      def setup
        @person = Person.new
      end

      describe "#name" do
        it "should return 'unknown' if no parameters are given" do
          assert_equal "unknown", @person.name
        end

        it "should be editable" do
          assert_equal "Johnny", @person.name = "Johnny"
        end
      end

      describe "#age" do
        it "should return 'unknown' if no parameters are given" do
          assert_equal "unknown", @person.age
        end

        it "should be editable" do
          assert_equal 15, @person.age = 15
        end
      end

      describe "#occupation" do
        it "should return 'unknown' if no parameters are given" do
          assert_equal "unknown", @person.occupation
        end

        it "should be editable" do
          assert_equal "English Teacher", @person.occupation = "English Teacher"
        end
      end
    end

    describe "when parameters are given" do
      def setup
        @person = Person.new('Nelly', 26, 'Teacher')
      end

      describe "#name" do
        it "should return 'Nelly'" do
          assert_equal "Nelly", @person.name
        end

        it "should be editable" do
          assert_equal "Corey", @person.name = "Corey"
        end
      end

      describe "#age" do
        it "should return the age" do
          assert_equal 26, @person.age
        end

        it "should be editable" do
          assert_equal 27, @person.age = 27
        end
      end

      describe "#occupation" do
        it "should return the occupation" do
          assert_equal "Teacher", @person.occupation
        end

        it "should be editable" do
          assert_equal "English Teacher", @person.occupation = "English Teacher"
        end
      end
    end
  end
end
```
```ruby
# rspec_person_spec.rb

require 'rspec'
require './person.rb'

describe Person do
  describe "when no parameters are given" do
    before(:each) do
      @person = Person.new
    end

    describe "#name" do
      it "should return 'unknown' if no parameters are given" do
        expect(@person.name).to eq('unknown')
      end

      it "should be editable" do
        expect(@person.name = "Johnny").to eq("Johnny")
      end
    end

    describe "#age" do
      it "should return 'unknown' if no parameters are given" do
        expect(@person.age).to eq("unknown")
      end

      it "should be editable" do
        (@person.age = 15).should be 15
      end
    end

    describe "#occupation" do
      it "should return 'unknown' if no parameters are given" do
        @person.occupation.should eq "unknown"
      end

      it "should be editable" do
        expect(@person.occupation = "English Teacher").to eq("English Teacher")
      end
    end
  end

  describe "when parameters are given" do
    before(:each) do
      @person = Person.new('Nelly', 26, 'Teacher')
    end

    describe "#name" do
      it "should return 'Nelly'" do
        @person.name.should eq "Nelly"
      end

      it "should be editable" do
        expect(@person.name = "Corey").to eq "Corey"
      end
    end

    describe "#age" do
      it "should return the age" do
        @person.age.should eq 26
      end

      it "should be editable" do
        expect(@person.age = 27).to eq 27
      end
    end

    describe "#occupation" do
      it "should return the occupation" do
        @person.occupation.should eq "Teacher"
      end

      it "should be editable" do
        expect(@person.occupation = "English Teacher").to eq "English Teacher"
      end
    end
  end
end
```
**Challenge**: Author your own Codewars kata