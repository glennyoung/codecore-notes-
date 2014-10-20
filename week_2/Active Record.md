# Active Record  
  
**What is a model?**  
When I try to model my app into objects, I used these classes to represent the different entities in my application. When we want to map the database, we use an ORM. We used DataMapper in Sinatra. In Rails, we use ActiveRecord.  
  
Our Question and Answer app is going to have two models: Question and Answer.  
  
What kind of information do I need in the Question model?  
  * id
  * title
  * description   

And in the Answer model?  
  * id
  * body  

Let's start by creating the Question model. Active Record's approach to things is a little different from DataMapper. The way to generate a model is using `rails generate [model-name]`. For our controllers, we used plural names, however for models we use singular. The table will be plural. So, if we make a model called 'Task' the database table will be 'tasks'. Rails does this for us.  
  
We can add the attributes to our model in the command line, like so  
```ruby
rails generate model question title:string description:text
```  
or a shortcut:
```ruby
rails g model question title:string description:text
```
***note***: I do not have to explicitly say, I need an id. One will be created automatically.  
***noteII***: The opposite of 'generate' is destroy. So if I wanted to destroy this model, I would use `rails destroy model question`   
  
Open up db/migrate/[migration-file]  
```ruby
class CreateQuestions < ActiveRecord::Migration
  def change
    create_table :questions do |t|
      t.string :title
      t.text :description

      t.timestamps
    end
  end
end
```  
  
Before running a `rake db:migrate` we could add different fields to the migration file, and this will create them when it runs the migration. For eample:  
```ruby
class CreateQuestions < ActiveRecord::Migration
  def change
    create_table :questions do |t|
      t.string :title
      # t.text :description, default: "no description"
      t.text :description
      t.integer :view_count             # add a view count field

      t.timestamps
    end
    add_index :questions, :title        # adding an index speeds up queries
  end
end
```  
We can put validations in our app for what data is stored in the database. Some teams may have server-side validation as well.  
  
Run `rake db:migrate` to migrate. This will also create a db/schema.rb file which shows the table structure for the database. Some more [Active Record migration](http://guides.rubyonrails.org/migrations.html) commands `rake db:create`, `rake db:migrate`, `rake db:rollback`, `rake db:reset`, `rake db:migrate:reset`.  
  
```ruby
db:create           # creates the database for the current env
db:create:all       # creates the databases for all envs
db:drop             # drops the database for the current env
db:drop:all         # drops the databases for all envs
db:migrate          # runs migrations for the current env that have not run yet
db:migrate:up       # runs one specific migration
db:migrate:down     # rolls back one specific migration
db:migrate:status   # shows current migration status
db:migrate:rollback # rolls back the last migration
db:forward          # advances the current schema version to the next one
db:seed             # (only) runs the db/seed.rb file
db:schema:load      # loads the schema into the current env's database
db:schema:dump      # dumps the current env's schema (and seems to create the db as well)

db:setup            # runs db:schema:load, db:seed

db:reset            # runs db:drop db:setup
db:migrate:redo     # runs (db:migrate:down db:migrate:up) or (db:migrate:rollback db:migrate:migrate) depending on the specified migration
db:migrate:reset    # runs db:drop db:create db:migrate
```  
  
To add a migration that adds something to a table, you can do something like  
```ruby
rails generate migration add_like_count_to_questions
```  
Open up your migration file, and add a column  
```ruby
class AddLikeCountToQuestions < ActiveRecord::Migration
  def change
    add_column :questions, :like_count, :integer
  end
end
```  
Add a migration to remove like_count from question `rails generate migration remove_like_count_from_questions`  
Open up the migration and add a line to remove the column  
```ruby
class RemoveLikeCountFromQuestions < ActiveRecord::Migration
  def change
    remove_column :questions, :like_count, :integer
  end
end
```  
***Note***: If you just made a mistake, you can rollback, make the fix, and delete that migration. However, if you are working on an app that is in production, or working with a team, you always want to fix forward, ie: migrate to add or remove columns, etc.  
  
## Rails Console  
To start the rails console, you can do `rails c` or `rails console`. This is similar to irb, but you use it while in your app's directory, and it has access to your rails app.  
  
Let's hope into rails console, and create a new question.  
```bash
rails c  
q = Question.new
=> #<Question id: nil, title: nil, description: nil, created_at: nil, updated_at: nil>

# add title, and description
q.title = "My first question title from console."
q.description = "Here is a fine description of this question that I am apparently not asking."
q
 => #<Question id: nil, title: "My first question title from console.", description: "Here is a fine description of this question that I ...", created_at: nil, updated_at: nil>
```  
To see if an object has gone into the database yet or not, you can use the `persisted?` method. try `q.persisted?`. To save into the databse, run `q.save`. Then check what the output of `q` is.  
```bash
q.save
q
 => #<Question id: 1, title: "My first question title from console.", description: "Here is a fine description of this question that I ...", created_at: "2014-03-25 17:16:46", updated_at: "2014-03-25 17:16:46">
 ```  
 
 Now, if we add some information to it, and save it again, our `created_at` and `updated_at` will be different.  
 ```bash
 q.description = "What is the output of the question now?"
 q.save
 q
  => #<Question id: 1, title: "My first question title from console.", description: "What is the output of the question now?", created_at: "2014-03-25 17:16:46", updated_at: "2014-03-25 17:18:39">
```  
We can also pass in a has of parameters when called Question.new  
```bash
q2 = Question.new(title: "Another question for you", description: "Do I have butterflies in my stomach all the time, because I'm super excited about everything, or because the world is constantly falling?")
q2.save
q2
 => #<Question id: 2, title: "Another question for you", description: "Do I have butterflies in my stomach all the time, b...", created_at: "2014-03-25 17:20:27", updated_at: "2014-03-25 17:20:27">
 ```   
To create without doing `.new` and `.save`, we can use `.create`  

```bash
Question.create(title: "I have a question.", description: "How many times have you given an egg to a raccoon?")
```  
## Add data validation  
Open up the question model and add `validates_presence_of :title`  
```ruby
# app/models/question.rb
class Question < ActiveRecord::Base

  validates_presence_of :title
  
end
```  
Then try saving a question in rails c without a title.  
```rake
# in rails console
reload!
q = Question.new
q.save
   (0.3ms)  BEGIN
[deprecated] I18n.enforce_available_locales will default to true in the future. If you really want to skip validation of your locale you can set I18n.enforce_available_locales = false to avoid this message.
   (0.4ms)  ROLLBACK
 => false 


# then check the errors!
q.errors
 => #<ActiveModel::Errors:0x000001022ebbb8 @base=#<Question id: nil, title: nil, description: nil, created_at: nil, updated_at: nil>, @messages={:title=>["can't be blank"]}> 
```  
[Validation reference](http://guides.rubyonrails.org/v3.2.13/active_record_validations_callbacks.html)  
  
Now add validation for the title to be unique using an alternative syntax that allows for more attributes.  
```ruby
# app/models/question.rb
class Question < ActiveRecord::Base
 
  # validates_presence_of :title
  validates :title, presence: true, uniqueness: true
  
  validates_presence_of :description, message: "must be present"
  
end
```   
To update the attributes of a record in the database, use `update_attributes`  
```bash
q.update_attributes(title: "updated title", description: "some new description")
```  
Some other class methods  
```bash
Question.first
Question.last
Question.all
# Question.destroy_all        # will delete all records
Question.find_by_title "abc"
```  
To have your data records display nicely in the rails console, you can use the gem hirb. Since you only need it for development, add it to a group in your Gemfile  
```ruby
# Gemfile
#...

group :development do
  gem 'hirb'
  gem 'interactive_editor'
  gem 'awesome_print'
end

#...
```  
After adding to your Gemfile, do a `bundle install` in the terminal, then in `rails c` do `Hirb.enable`. Add the following to your .irbrc dotfile. This will require and enable 'hirb' when irb (or rails c) loads, and if not, it will give an error.  
```ruby
# ~/.irbrc 

#...

begin
  require 'hirb'
  Hirb.enable
rescue LoadError => err
  warn "Couldn't load hirb: #{err}"
end

#...

```  
Try some more class methods  
```bash
Question.select(:id, :title, :description)
Question.select(:id, :title, :description).limit(2)
Question.select(:id, :title, :description).offset(2)
```  
## Basic Queries  
Queries are, in a large part, based on the `WHERE` statment.  
Here are some examples  
```bash
Question.where.not(title: "abc")                # will return all questions where the title is not equal to "abc"
Question.where(["title like ?", "%fas%"])       # I pass in an array to the WHERE. The first argument is the query string.
Question.where(["description like ?", "%fas%"])

# to find all records where title or description contains a string
Question.where(["title like ? OR description like ?", "%fas%", "%fas%"])

Question.where(["title like ? OR description like ?", "%title%", "%what%"]).where(["created_at > ?", 10.days.ago])

Question.order("title ASC")
Question.order("title DESC")
```   
Adding limits to the model, using [scopes](http://api.rubyonrails.org/classes/ActiveRecord/Scoping/Named/ClassMethods.html#method-i-scope)  
```ruby
# app/models/question.rb
class Question < ActiveRecord::Base

  #validates_presence_of :title
  validates :title, presence: {message: "must be there"}  , uniqueness: true

  validates_presence_of :description, message: "must be present"
  
  # a default scope will be used for al queries
  default_scope order("title ASC")

  # "->" is shorthand for lambda
  # to pass in a variable, use "->(x)"
  # scope :recent_tn, lambda { order("created_at DESC").limit(10) }
  
  scope :recent, lambda {|x| order("created_at DESC").limit(x) }
  scope :recent_ten, -> { order("created_at DESC").limit(10) }
  
  # this can be shorted by writing a scope
  def self.recent_ten
    order("created_at DESC").limit(10)
  end
  
  def self.recent(x)
    order("created_at DESC").limit(x)
  end
  
end
```  
## Callbacks  
Callbacks are widely used in Active Record (see: [Active Record Callbacks](http://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html)). Sometimes I want to do an operation on something before I add it to a record, and I want to do it to every record in the databse. Let's say I want to capitalize the title before i save it in the database, I can do something like this:  
```ruby
# app/models/question.rb
class Question < ActiveRecord::Base

  #validates_presence_of :title
  validates :title, presence: {message: "must be there"}  , uniqueness: true

  validates_presence_of :description, message: "must be present"
  
  # a default scope will be used for al queries
  default_scope order("title ASC")

  # "->" is shorthand for lambda
  # to pass in a variable, use "->(x)"
  # scope :recent_tn, lambda { order("created_at DESC").limit(10) }
  
  scope :recent, lambda {|x| order("created_at DESC").limit(x) }
  scope :recent_ten, -> { order("created_at DESC").limit(10) }
  
  # this can be shorted by writing a scope
  def self.recent_ten
    order("created_at DESC").limit(10)
  end
  
  def self.recent(x)
    order("created_at DESC").limit(x)
  end
  
  before_save :capitalize_title             # call the before_save action :capitalize_title
  
  private
  
  def capitalize_title                 # create a method to capitalize the title before saving
    self.title.capitalize!
  end
  
end
```  