# Rails: One to Many  

*note*: Add [form validation](http://getbootstrap.com/css/#forms-control-validation) with bootstrap classes.  


```ruby
# config/routes.rb
# Nesting three levels deep is bad
resources :projects do
  resources :discussions do
    resources :comments
  end
end

# This will give cleaner routs
resources :projects do
  resources :discussions
end

resources :discussions do
  resources :comments
end

```  
  
```ruby
resources :answers, only: [] do
  resources :comments
end

resources :questions do
  resources :answers
  member do
    post :vote_up
    post :vote_down
  end
  post :search, on: :collection
end
```  
  
If you want localhost:3000/asjfadsjfa to link to questions index, you could  
```ruby
get "/asjfadsjfa" => "questions#index" #

```  
  
## Add delete button for every answer  
 
Our routes show the question asnwer path as `/questions/:question_id/answers/:id(.:format)`. So, we can pass in the @question and @answer variables using the `question_answers_path` helper method.  
```haml
/app/views/show.html.haml
# ...
    %hr
    -@question.answers.each do |answer|
      .well
        .row
          .col-sm-8.col-md-8.col-xs-8
            %p= answer.body
            %p Created on #{formatted_date(answer.created_at)}
          .col-sm-8.col-md-8.col-xs-8
            .pull-right= button_to "Delete", question_answers_path(@question, @answer), method: :delete, class: "btn btn-danger", data: {confirm: "Are you sure you want to delete the answer?"}
```  
Add a method to delete in the answers controller  
```ruby
# answers_controller.rb
class AnswersController < ApplicationController
  before_action :find_question

  def create
    @question = Question.find params[:question_id]
    @answer = @question.answers.new(answer_attributes)
    if @answer.save
      redirect_to @question, notice: "Answer created successfully."
    else
      render "/questions/show"
    end
  end

  def destroy
    @answer = @questions.answers.find(params[:id])
    if @answer.destroy
      redirect_to @question, notice: "Answer deleted"
    else
      redirect_to @question, error: "We had trouble deleting the answer"
    end
  end

  private

  def answer_attributes
    params.require(:answer).permit([:body])
  end

  def find_question
    @question = Question.find params[:question_id]
  end
end
```  

We can use the well that displays our answers as a partial. To do so, simply copy it out of our questions/show page, and create a file under answers that begins with an underscore.
```haml
# views/answers/_answer.html.haml
.well
  .row
    .col-sm-8.col-md-8.col-xs-8
      %p= answer.body
      %p Created on #{formatted_date(answer.created_at)}
    .col-sm-4.col-md-4.col-xs-4
      .pull-right= button_to "Delete", question_answer_path(@question, @answer), method: :delete, class: "btn btn-danger", data: {confirm: "Are you sure you want to delete the answer?"}
```  
Then reference that partial in the questions page.  
```haml
/ views/questions/show.html.haml

  %hr
  -@question.answers.each do |answer|
  = render "/answers/answer", answer: answer
```  
Rails has a shortcut for this, provided all the variables and files are following the same naming scheme, as we have in our case.  
```haml
%hr
= render @answers
```   

We can change the AnswersController to inherit from the QuestionsController so then answers have access to the methods available to question.  
```ruby
class AnswersController < QuestionsController
  before_action :find_question

  def create
    @question = Question.find params[:question_id]
    @answer = @question.answers.new(answer_attributes)
    if @answer.save
      redirect_to @question, notice: "Answer created successfully."
    else
      render "/questions/show"
    end
  end

  def destroy
    @answer = @questions.answers.find(params[:id])
    if @answer.destroy
      redirect_to @question, notice: "Answer deleted"
    else
      redirect_to @question, error: "We had trouble deleting the answer"
    end
  end


  private

  def answer_attributes
    params.require(:answer).permit([:body])
  end

end
```  
We no longer need a private method to find a question in the AnswersController, and can instead add an or by id to the find question method in the QuestionsController.  
```ruby
# app/controllers/questions_controller.rb
#...

  def find_question
    @question = Question.find(params[:question_id] || params[:id])
  end
```
## Comments  
Add a comment model `rails generate resource comment body:text answer:references`   
Add a has many to the answer model and make it `dependent: :destroy` so we don't have any orphaned information left in the database.    
```ruby
# app/models/answer.rb
class Answer < ActiveRecord::Base
  belongs_to :question
  has_many :comments, dependent: :destroy

  validates_presence_of :body

  scope :ordered_by_creation, -> { order("created_at DESC")}
end
```  
Test it out in rails c  
```ruby
a = Answer.find 11
a.comments.create(body: "asdfasdf")
```  
Let's add `better_errors` and `binding_of_caller` gems to your development group:   
```
# Gemfile

# ...

group :development do
  gem 'better_errors'
  gem 'binding_of_caller'
  gem 'hirb'
  gem 'interactive_editor'
  gem 'awesome_print'
  gem 'quiet_assets'
end

#...
```  
## Rails: Has One  
   
We'll use a seperate model to keep things clean.  
```bash
rails generate model question_detail notes:text question:references
```  

Now if we go to QuestionDetail.rb, we see it belongs to question  
```ruby
class QuestionDetail < ActiveRecord::Base
  belongs_to :question
end
```  
add `has_one :question_detail` to the question model.  
  
Now, if we go to `rails console` we can do some stuff that we can only do with a `has_one`    
```ruby
  # has_one
  question.build_question_detail(notes: "asdjklasj")
  question.create_question_detail(notes: "klajsdlks")
  
  # has_many
  question.answers.build(notes: "Hey")
  question.answers.create(notes: "Hey")

```  
Some rails c stuff
```
q = Question.first
qd = q.build_question_detail
qd.notes = "kajsdkjasdkjhsa"
qd
qd.save
```