# RSpec  
Unit Test ships with Rails. We will be using RSpec.  
[RSpec vs. TestUnit](http://www.rubyinside.com/dhh-offended-by-rspec-debate-4610.html) |  
  
Let's start by installing the RSpec gem.  
```ruby
# Gemfile

# ...

gem 'rspec-rails', group: [:development, :test]

# ...  
```
Run bundle install, then install rspec on the command line using `rails g rspec:install`  
***note*** if you want to start a project without the test folder, you can by using `rails new my_app -T`.  
  
Let's just destroy our comment model, then recreate it using TDD.  
```ruby
rails destroy model comment
rails generate model comment body:text answer:references
```  
This generate a migration file, a model and an rspec spec.   
```ruby
# spec/models/comment_spec.rb
require 'spec_helper'      

  describe Comment do        
    pending "add some examples to (or delete) #{__FILE__}"
  end 
```  
RSpec using some words like 'describe' and 'context' to say it's going to do a 'Test'.  
  
We can delete the pending default test, then we'll try making a test.  
```ruby
require 'spec_helper'      
                           
describe Comment do        
  it "doesn't allow creating comment without a body" do
    comment = Comment.new
    expect(comment).to_not be_valid
  end
end
```  
Then make a new comment in rails c.  
```ruby
comment = Comment.new
comment.valid?
```  
Get the test ready with `rake db:test:prepare`. On versions of rails newer than 4.1.0 this is done automatically and running it will only give a deprecation warning. Then run the test in terminal with `rspec spec`. This should give you a failing test! Because, our comment model can be created without a body. Now, we can open our comment model, and add validation.  
```ruby
# app/models/comment.rb
class Comment < ActiveRecord::Base
  validates_presence_of :body
  
  belongs_to :answer
end
```  
Let's add another test to make sure the length of the body is at least three lines.  
```ruby
require 'spec_helper'

describe Comment do

  it "doesn't allow creating comment with no content" do
    comment = Comment.new
    expect(comment).to_not be_valid
  end

  it "doesn't allow creating comment without a body" do
    comment = Comment.new
    expect(comment).to_not be_valid
    comment.save
    comment.errors.messages.should have_key(:body)
  end

  it "doesn't alow comment body shorter than 3 characters" do
    comment = Comment.new(body: "ab")
    expect(comment).to_not be_valid
  end
end
```  
You can run just the test that you want to check by specifying the path and the line number. In this case, my new test is on line 17, so to see this test fail, I can do
```ruby
rspec spec/models/comment_spec.rb:17
```  
Then to make this test past I can add to the validation in the comment model
```ruby
# app/models/comment.rb
class Comment < ActiveRecord::Base
  validates :body, presence: true, length: {minimum: 3}

  belongs_to :answer
end
```  
1.) Write tests, and then add a method to cleans white space when a user adds a comment.  
```ruby
# spec/models/comment_spec.rb

# ...

  describe ".sanitize" do
    it "removes repeated white spaces in the body" do
      text = "body  with     extra          spaces."
      comment = Comment.new(body: text)
      comment.sanitize
      expect(comment.body).to eq("body with extra spaces.")
    end
  end
  
# ...
```  
This test should fail, because there is no method defined called 'sanitize'. We can add the minimal amount of code to make this pass.  
```ruby
# app/models/comment.rb

# ...

def sanitize
end

# ...  
```
Now, our test fails, but for a different reseason. The result doesn't match what we are expecting. Now, we can add to our sanitize method.  
```ruby 
# app/models/comment.rb

# ...


  def sanitize
    self.body.squeeze!(" ")
  end


# ...  
```  
Now, we can test to make sure it strips out spaces before and after the string.  
```ruby  
# spec/models/comment_spec.rb

# ...

  describe ".sanitize" do
    it "removes repeated white spaces in the body" do
      text = "body  with     extra          spaces."
      comment = Comment.new(body: text)
      comment.sanitize
      expect(comment.body).to eq("body with extra spaces.")
    end

    it "strips spaces at the edges of the body text" do
      text = "     body  with     extra          spaces.    "
      comment = Comment.new(body: text)
      comment.sanitize
      expect(comment.body).to eq("body with extra spaces.")
    end
  end
  
# ...
```  
And to make this test pass, we can add strip! to our sanitize method  
```ruby
# app/models/comment.rb

# ...


  def sanitize
    self.body.squeeze!(" ").strip!
  end


# ...  
```  
***Note***: If you are having troubles with this returning Nil, try using `self.body = body.sqeeze(" ").strip` instead.  
  
Add a new describe to the spec to test for comments  
```ruby
# spec/models/comment_spec.rb

  # ...
  
  describe "Recent Ten Comments Scode" do

    before do
      11.times {|x| Comment.create!(body: "valid body #{x}")}
    end

    it "returns 10 records" do
      expect(Comment.recent_ten.count).to eq(10)
    end

    it "returns most recent" do

    end

  end
  
  # ...
  
```  
Add a scope to the comment model to make the test pass  
```ruby
# app/models/comment.rb

# ...

  scope :recent_ten, -> {limit(10)}
  
# ...
```  
Add test logic to 'returns most recent'  
```ruby
# spec/models/comment_spec.rb

# ...

    it "returns most recent" do
      expect(Comment.recent_ten).to_not include(Comment.first)
    end
    
# ...
```  
Then make it pass by adding to the scope in your comment model.  
```ruby
# spec/models/comment.rb

# ...

 scope :recent_ten, -> {limit(10).order("created_at DESC")}
 
# ...
```
2.) Test Drive: Make sure answers are capitalized before inserting into the database.  
```ruby
# spec/models/answer_spec.rb

require 'spec_helper'

describe Answer do

  it "capitalizes answers before saving in the database" do
    answer = Answer.new
    answer.body = "my funny answer"
    answer.save
    expect(answer.body).to eq("My funny answer")
  end
end

```   

Then make it pass by adding a before save in the answer model.  
```ruby
# app/models/answer.rb
class Answer < ActiveRecord::Base
  before_save :cap_body
  
  # ...

  def cap_body
    self.body.capitalize!
  end
  
  # ...

end

```  
3.) Test Drive: Full name method in user that returns first name and last name or email  
```ruby
# spec/models/user_spec.rb

require "spec_helper"

describe User do
  
  describe ".full_name" do
    it "includes first_name if it exists" do
      user = User.new(first_name: "Tam",
                      last_name: "Kbeili",
                      email: "tam@codecore.ca")
      expect(user.full_name).to include("Tam")
    end
    
    it "includes last_name if it exists" do
      user = User.new(first_name: "Tam",
                      last_name: "Kbeili",
                      email: "tam@codecore.ca")
      expect(user.full_name).to include("Kbeili")
    end
  end

end

```  
Let's use [Factory Girl](https://github.com/thoughtbot/factory_girl/blob/master/GETTING_STARTED.md) to generate models automatically for us. In your Gemfile add `gem "factory_girl_rails"` to your test and development group. Then bundle install.  
  
Add to your spec helper `config.include FactoryGirl::Syntax::Methods`

Create a new factory in the terminal `rails g factory_girl:model comment`  
Then open it up and add a field for body, with a call to Faker to create random sentences
```ruby
# spec/factories/comments.rb
FactoryGirl.define do    
  factory :comment do
  body Faker::Lorem.sentence
  end
end
```
  
Then inside rails console, you can try FactoryGirl.create(:comment)
```
18.times { FactoryGirl.create(:comment) }
```
  
Create an Answer factory
```ruby
rails g factory_girl:model answer
```  
Open it and add some logic for association and body
```ruby
FactoryGirl.define do
  factory :answer do
    association :question, factory: :question
    body Faker::Company.bs
  end
end
```  
You can override the factory default by setting it when calling the factory  
```ruby
FactoryGirl.create(:comment, body: "lol")

# Add comments to a particular answer
answer = Answer.last
FactoryGirl.create(:comment, answer: answer)
```