How do I add a link to my homepage in order to take me to a form where I can create my new question?
  
```erb
# app/views/questions/index
 <h1>Listing All Questions</h1>
 
 <%= link_to "Create New Question", new_question_path %>
 
  <% @questions.each do |question| %>
 
  <h2><%= link_to question.title, question_path(question) %></h2>
 
  <p><%= question.description %></p>
 
  <p>Created On: <%= question.created_at.strftime("%Y-%b-%d") %></p>
 
  <hr>
 
 <% end %> 
```  
If I want to show the actual question, I should change my questions controller. What should I do? And let's create a way to click on a specific question, where I can view the details of that question. And link back to the questions index page.  And even edit or delete that question.  
```ruby
#app/controllers/questions_controller.rb

# ...

def show
  @question = Question.find(params[:id])
end

# ...
```

```erb
#app/views/questions/show.html.erb

<h1><%= @question.title %></h1>
<p><%= @question.description %>
<p><%= @question.created_at.strftime("%Y-%b-%d") %>

<br>

<%= link_to "Edit", edit_question_path(@question) %>

<br>

<%= link_to "All Questions", questions_path %>

```  
Add a method in your questions_controller to edit.  
```ruby
# app/controllers/questions_controller.rb

#...

def edit
  @question = Question.find(params[:id])
end

#...


```
Edit Page  
```erb
# app/views/questions/edit.html/erb
<h1>Editing Question</h1>

<%= form_for @question do |f| %>
  
  <div class="form-field">
    <%= f.label :title, "TitLe" %>
    <%= f.text_field :title, class: "form-control" %>
  </div>

  <div class="form-field">
    <%= f.label :description %>
    <%= f.text_area :description, class: "form-control" %>
  </div>

  <%= f.submit %>

<% end %>
```  
Rather than repeat the same thing to instantiate a question in each of your methods. It's a good idea to perform something called a 'before action' which will instantiate a question object for you.  

```ruby
class QuestionsController < ApplicationController
  before_action :find_question, 
                  only: [:show, :edit, :destroy, :update]

  def index
    @questions = Question.all
  end

  def create
    #@question = Question.new(params[:question]) # We used to be able to do this, but there were some security issues.
    # now, in Rails 4, the default action is to prevent everything, rather than allowing. 
    question_attributes = params.require(:question).permit([:title, :description])
    @question = Question.new(question_attributes)

    if @question.save
      redirect_to questions_path, notice: "Your question was created successfully."
    else
      flash.now[:error] = "Please correct the form"
      render :new
    end
  end
  
  def new
    @question = Question.new
  end

  def show
  end

  def edit
  end
  
  def update
  end

  def destroy
    render text: "Question: #{params[:id]} has been successfully deleted."
  end
  
  private
  
  def find_question
    @question = Question.find(params[:id])
  end

end
```  
Also, rather than copy/pasting our form to mulitple pages, we can create what's called a partial (note partial filenames begin with an underscore "_").  
```erb
# app/views/questions/_form.html.erb
<%= form_for @question do |f| %>
  
  <div class="form-field">
    <%= f.label :title, "TitLe" %>
    <%= f.text_field :title, class: "form-control" %>
  </div>

  <div class="form-field">
    <%= f.label :description %>
    <%= f.text_area :description, class: "form-control" %>
  </div>

  <%= f.submit %>

<% end %>

```  
Then, we can call this form partial in a view file with 'render'  
```erb
# app/views/questions/new.html.erb

<h1>Create New Question</h1>

<%= render 'form' %>
```   
```erb
# app/views/questions/edit.html.erb

<h1>Edit Question</h1>

<%= render 'form' %>
```  
If you want to use a partial from another folder, you will need to use the full path starting from views. For example, views/questions/_form.html.erb.  
  
How can we get a different label on the button, based on wether the question is in the database or not?   

`<%= f.submit (@question.persisted? ? "Update" : "Save"), class: "btn btn-default" %>`  
  
To update a question in our databse through a form, we will create a private method question_attributes  
```ruby
# app/controllers/questions_controller.rb

# ...
def update
  if @question.update_attributes(question_attributes)
    redirect_to @question, notice: "Question updated successfully"
  else
    flash.now[:error] = "Couldn't update!"
    render :edit
  end
end

private

def question_attributes
  params.require(:question).permit([:title, :description])
end

# ...
```  
Let's look now at the destroy action. What do I do to destroy a record in the database?  
```ruby
# app/controllers/questions_controller.rb

#... 

def destroy
  if @question.destroy
    redirect_to questions_path, notice: "Question deleted successfully."
  else
    redirect_to question_path, error: "We had trouble deleting."
  end
end
# ...  
```  
Then add a link to delete on the show page.  
```erb
# app/views/questions/show.html.erb
<h1><%= @question.title %></h1>
<p><%= @question.description %>
<p><%= @question.created_at.strftime("%Y-%b-%d") %>

<br>
<%= link_to "Edit", edit_question_path(@question) %>

<%= link_to "Delete", @question, method: :delete, data: { confirm: "Are you sure you want to delete this question?" } %>
<br>

<%= link_to "All Questions", questions_path %>


```  
## Adding Votes  
You can specify the field in the migration, like so: `rails g migration add_vote_count_to_questions vote_count:integer`  
This creates the field for us in our migration file  
```ruby
# db/migrate/201403279827982374_add_vote_count_to_questions.rb
class AddVoteCountToQuestions < ActiveRecord::Migration
  def change
    add_column :questions, :vote_count, :integer
  end
end
```  
If you want to give it a default of 0, you can do so manually.  
```ruby
# db/migrate/201403279827982374_add_vote_count_to_questions.rb
class AddVoteCountToQuestions < ActiveRecord::Migration
  def change
    add_column :questions, :vote_count, :integer, default: 0
  end
end
```  
Run `rake db:migrate` Then add a vote up link to your show page  
```erb
#app/views/questions/show.html.erb

<h1><%= @question.title %></h1>
<p><%= @question.description %>
<p><%= @question.created_at.strftime("%Y-%b-%d") %>

<br>
  <%= link_to "Vote Up", vote_up_question_path(@question) %>

<br>
<%= link_to "Edit", edit_question_path(@question) %>

<%= button_to "Delete", @question, method: :delete, data: { confirm: "Are you sure you want to delete this question?" }, class: "btn btn-default" %>
<br>

<%= link_to "All Questions", questions_path %>
```  

Add some methods in your questions controller to vote up and vote down  
```ruby
# app/controllers/questions_controller.rb

class QuestionsController < ApplicationController
  before_action :find_question, 
                  only: [:show, :edit, :destroy, :update, :vote_up, :vote_down]

  def index
    @questions = Question.all
  end

  def new
    @question = Question.new
  end

  def create
    #@question = Question.new(params[:question]) # We used to be able to do this, but there were some security issues.
    # now, in Rails 4, the default action is to prevent everything, rather than allowing. 
    question_attributes = params.require(:question).permit([:title, :description])
    @question = Question.new(question_attributes)

    if @question.save
      redirect_to questions_path, notice: "Your question was created successfully."
    else
      flash.now[:error] = "Please correct the form"
      render :new
    end
  end

  def show
  end

  def edit
  end

  def update
    if @question.update_attributes(question_attributes)
      redirect_to @question, notice: "Question updated successfully"
    else
      flash.now[:error] = "Couldn't update!"
      render :edit
    end
  end

  def destroy
    if @question.destroy
      redirect_to questions_path, notice: "Question deleted successfully."
    else
      redirect_to question_path, error: "We had trouble deleting."
    end
  end
  
  def vote_up
    @question.increment!(:vote_count)
    session[:has_voted] = true
    redirect_to @question
  end
  
  def vote_down
  end
  
  def search
  end

  private

  def question_attributes
    params.require(:question).permit([:title, :description])
  end

  def find_question
    @question = Question.find(params[:id])
  end

end

```

Then inside the show, add a vote count  
```ruby
```  
In our routes.rb the default for vote up on our show path is get. We could make it a button, or if we want to keep it a link, we could add `method: :post`.  
```erb
# app/views/questions/show.html.erb
<h1><%= @question.title %></h1>
<p><%= @question.description %>
<p><%= @question.created_at.strftime("%Y-%b-%d") %>

<p>Vote Count: <%= @question.vote_count %>
<br>
  <% if session[:has_voted] %>
    You voted already!
    <% else %>
  <%= button_to "Vote Up", vote_up_question_path(@question) %>
  <% end %>

<br>
<%= link_to "Edit", edit_question_path(@question) %>

<%= button_to "Delete", @question, method: :delete, data: { confirm: "Are you sure you want to delete this question?" }, class: "btn btn-default" %>
<br>

<%= link_to "All Questions", questions_path %>
```  
## Adding Helper Methods  
We can use helper modules. When we use `rails generate controller`, it automatically puts a helper for every controller in the app/helpers directory.  
```ruby
# app/helpers/application_helper.rb
module ApplicationHelper
  def formatted_date(date)
    date.strftime("%Y-%B-%d")
  end
end
```  
Then in the index view  
```erb
<h1>Listing All Questions</h1>

<%= link_to "Create a New Question", new_question_path %>

<% @questions.each do |question| %>

    <h2><%= link_to question.title, question_path(question) %></h2>
    <p><%= question.description %></p>
    <p><p>Created On: <%= formatted_date(@question.created_at) %></p>
    <hr>

<% end %>
```  
and in the show page
```erb
<h1><%= @question.title %></h1>
<p><%= @question.description %>
<p><%= formatted_date(@question.created_at) %>

<br>
  <% if session[:has_voted] %>
    You voted already! 
    <% else %>
  <%= button_to "Vote Up", vote_up_question_path(@question) %>
    <% end %>

<br>
<%= link_to "Edit", edit_question_path(@question) %>

<%= button_to "Delete", @question, method: :delete, data: { confirm: "Are you sure you want to delete this question?" }, class: "btn btn-default" %>
<br>

<%= link_to "All Questions", questions_path %>

```

### Available Callbacks  (ActiveRecord::Callbacks::CALLBACKS)
```bash
:after_initialize
:after_find
:after_touch
:before_validation
:after_validation
:before_save
:around_save
:after_save
:before_create
:around_create
:after_create
:before_update
:around_update
:after_update
:before_destroy
:around_destroy
:after_destroy
:after_commit
:after_rollback
```  
**Bonus**: Rather than setting the default values for vote_count to 0 in the migration file, we could add an `after_initialize` action to the model.  

```ruby
# app/models/question.rb
#...

after_intitialize :set_defaults

private

def set_defaults
  self.vote_count ||= 0
end
```  