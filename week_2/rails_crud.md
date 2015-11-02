# Building a Rails CRUD

<!-- Adding a line to rails_crud   -->

For the questions index page create an instance variable with all questions.  

```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController

  def index
    @questions = Question.all
  end

  def create
    render text: "Create a question"
  end

  def show
  end

  def edit
    render text: "Editing question: #{params[:id]}"
  end

  def new
    render text: "A new question"
  end

  def destroy
    render text: "Question: #{params[:id]} has been successfully deleted."
  end

end
```  

On the index.html.erb set some items to display each question, including title, description, and created_at using Ruby's [strftime](http://www.ruby-doc.org/core-2.1.1/Time.html#method-i-strftime) method.  
```erb
# app/views/questions/index.html.erb
<h1>Listing All Questions</h1>

<% @questions.each do |question| %>

    <h2><%= question.title %></h2>
    <p><%= question.description %></p>
    <p>Created On: <%= question.created_at.strftime("%Y-%b-%d") %></p>
    <hr>

<% end %>

```  
Let's create a page where we can fill in a question title and description, and we can click a button to save it in the database. The first step is inside the new method of our questions_controller.rb, we will instantiate a question instance variable.  
```ruby
# app/controllers/questions_controller.rb  

# ...
def new
  @question = Question.new
end

# ...

```  
Now we need a view called 'new' to enter the information for our new question.  
```erb
# app/views/questions/new.html.erb  
<h1>New Question</h1>

<% form_for @question do |f| %>

  <%= f.label :title %>
  <%= f.text_field :title %>

  <%= f.lable :description %>
  <%= f.text_area :description %>

  <%= f.submit %>

<% end %>

```  
Params is a hash of hashes, some of its keys have hashes inside, auth.token, question {title: "...", description: "..."}. We can see these parameters when we submit a new question.  
```bash
Started POST "/questions" for 127.0.0.1 at 2014-03-26 13:47:37 -0700
Processing by QuestionsController#create as HTML
  Parameters: {"utf8"=>"✓", "authenticity_token"=>"w0WxTUZNQrbHnIHsuwfTYtJxKtYGzN3XlnOb88xc7qw=", "question"=>{"title"=>"Here's a question", "description"=>"kljsdlkajs ajsfd kljf;k djsaf"}, "commit"=>"Create Question"}
```  
In your questions controller add a method create that will render text showing the question title.    
```ruby
# app/controllers/questions_controller.rb  

# ...

def create
  render text: "Create..#{params[:question][:title]}"
end

# ...  

```
Now, to save this to the database, we will need to modify the create method. We no longer have access to the instance variable from the new request, so we need a new instance of question.  

```ruby
# app/controllers/questions_controller.rb  

# ...

def create
  @question = Question.new
  @question.title = params[:question][:title]
  @question.title = params[:question][:description]
  @question.save
  redirect_to questions_path
end

```  
Our logs have a lot of entries for accessing html/css files. We don't really care about those, so to remove them, you can just add `gem quiet_assets` to your Gemfile.  

Let's refactor that create method to be a little better.  
```ruby
# app/controllers/questions_controller.rb  

# ...

def create
  #@question = Question.new(params[:question]) # We used to be able to do this, but there were some security issues.
  # now, in Rails 4, the default action is to prevent everything, rather than allowing.
  question_attributes = params.require(:question).permit([:title, :description])
  @question = Question.new(question_attributes)

  if @question.save
    redirect_to questions_path, notice: "Your question was created successfully."
  else
  flash.now[:error] = "PLease correct the form"
    render :new
  end
end

```  
In IRB, try saving an instance variable question without the required params, then check the errors.  
```bash
q = Question.new
q.save
q.errors.any?
q.errors
q.errors.messages
```   
Let's add a way to display errors on our new question form:  
```erb
# app/views/questions/new.html.erb  
<% if @question.errors.any? %>

 <ul>
   <% @question.errors.full_messages.each do |message| %>
     <li><%= message %></li>
   <% end %>
  </ul>
<% end %>

<% form_for @question do |f| %>

  <%= f.label :title %>
  <%= f.text_field :title %>

  <%= f.lable :description %>
  <%= f.text_area :description %>

  <%= f.submit %>

<% end %>

```  
Set the flash notices to display through the application layout.  

```erb
#app/views/layouts/application.html.erb
<!DOCTYPE html>
<html>
<head>
  <title>AwesomeAnswers</title>
  <%= stylesheet_link_tag    "application", media: "all", "data-turbolinks-track" => true %>
  <%= javascript_include_tag "application", "data-turbolinks-track" => true %>
  <%= csrf_meta_tags %>
</head>
<body>
  <div class="container">

    <nav class="nav-main">
      <%= link_to "About Us", about_us_path, class: "btn btn-default nav-btn", id: "about" %>
      <%= link_to "FAQ", faq_path, class: "btn btn-default nav-btn" %>
      <%= link_to "Help", help_path, class: "btn btn-default nav-btn" %>
      <%= link_to "New Question", new_question_path, class: "btn btn-default nav-btn" %>
    </nav>

    <% if flash[:notice] || flash[:error] %>
      <h3><%= flash[:notice] || flash[:error] %></h3>
    <% end %>

    <%= yield %>



  </div>
</body>
</html>
```  
We can add private methods to our questions_controller.rb to clean this up a little, and define the method for  questions_attributes outisde the create method.  
```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController

  def index
    @questions = Question.all
  end

  def create
    #@question = Question.new(params[:question]) # We used to be able to do this, but there were some security issues.
    # now, in Rails 4, the default action is to prevent everything, rather than allowing.
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
    render text: "Editing question: #{params[:id]}"
  end

  def new
    @question = Question.new
  end

  def destroy
    render text: "Question: #{params[:id]} has been successfully deleted."
  end

  private

  def question_attributes
    question_attributes = params.require(:question).permit([:title, :description])
  end
end


```
