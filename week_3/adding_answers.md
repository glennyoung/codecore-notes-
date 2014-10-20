# Add Answers to Awesome Answers  
[Vestal Version](https://github.com/laserlemon/vestal_versions) |  
  
```bash
rails generate model answer body:text question:references
```  
**note**: If you are adding a reference to an existing project, you can make a migration like  
```bash
rails g migration add_project_references_to_tasks project:references
```
  
This gives the migration file  
```ruby
class CreateAnswers < ActiveRecord::Migration
  def change
    create_table :answers do |t|
      t.text :body
      t.references :question, index: true

      t.timestamps
    end
  end
end
```  
Run `bundle exec rake db:migrate` and open up the model: question.rb, where we will add `has_many`, and make it `dependent: :destroy` if you want the answers to be deleted if the question is.      
```ruby
# app/models/question.rb
class Question < ActiveRecord::Base
  has_many :answers, dependent: :destroy
  #validates_presence_of :title
  validates :title, presence: {message: "must be there"}  , uniqueness: true

  validates_presence_of :description, message: "must be present"

  # after_intitialize :set_defaults

  # a default scope will be used for al queries
  default_scope {order("title ASC")}

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
In the answer model, because we used `question:references` when we generated the model, we have `belongs_to`  
```ruby
class Answers < ActiveRecord::Base
  belongs_to :question
end
```  

In Rails Console, get a question instantiated, and give it an answer.  
```ruby
question = Question.last
answer1 = Answer.new(body: "This is my answer!")

question.answers << answer1
  
# We can chain other scopes with the association.  
question.answers.where(body: "something")
```  
*Generate a controller*  
```
rails generate controller answers
```  
Add routes for answers  
```ruby
# config/routes.rb
AwesomeAnswers::Application.routes.draw do

  resources :questions, only: [:new, :index, :create, :show, :edit, :destroy] do
    resources :answers
    member do
      post :vote_up
      post :vote_down
    end
    post :search, on: :collection
  end

end
```  
Check your [routes](localhost:3000/rails/info/routes).  
  
Add a form for answers on your questions show view  
```haml
    %h2 Add an Answer
    = form_for @answer, url: question_answers_path(@question)
      do |f|
      = f.text_area :body
      %br
      = f.submit "Submit an answer", class: "btn btn-primary"
```  
  
instantiate a new answer in the questions controller show method
```ruby
app/controllers/questions_controller.rb

  # ...
  def show
    @answer = Answer.new
  end
  # ...
```  
Give your answers controller a create method  
```ruby
class AnswersController < ApplicationController

  def create
    @question = Question.find params[:question_id]
    @answer = @question.answer.new(answer_attributes)
    if @answer.save
      redirect_to @question, notice: "Answer created successfully."
    else
      render "/questions/show"
    end
  end


  private

  def answer_attributes
    params.require(:answer).permit([:body])
  end

end
```  
Add validation to the answer model  
```ruby
class Answer < ActiveRecord::Base
  belongs_to :question

  validates_presence_of :body
end
```  
And add a way to display any errors on the questions show page  
```haml
%h1= @question.title
%p
  = @question.description
  %p
    = formatted_date(@question.created_at)
    = succeed "Vote" do
      %br/
    Count: #{@question.vote_count}
    %br/
    - if session[:has_voted]
      You voted already!
    - else
      = button_to "Vote Up", vote_up_question_path(@question)
    %br/
    = link_to "Edit", edit_question_path(@question)
    = button_to "Delete", @question, method: :delete, data: { confirm: "Are you sure you want to delete this question?" }, class: "btn btn-default"
    %br/
    = link_to "All Questions", questions_path

    %h2 Add an Answer
    - if @answer.errors.any?
      %ul
        - @answer.errors.full_messages.each do |message|
          %li= message

    = form_for @answer, url: question_answers_path(@question) do |f|
      = f.text_area :body
      %br
      = f.submit "Submit an answer", class: "btn btn-primary"
```  
Modify the show view to display the answers.  
```haml

    = form_for @answer, url: question_answers_path(@question) do |f|
      = f.text_area :body
      %br
      = f.submit "Submit an answer", class: "btn btn-primary"

    %hr
    -@question.answers.each do |answer|
      .well
        %p= answer.body
        %p Created on #{formatted_date(answer.created_at)}
```

To display the answers ordered by creation just make a scope in the answer model  
```ruby
class Answer < ActiveRecord::Base
  belongs_to :question

  validates_presence_of :body

  scope :ordered_by_creation, -> { order("created_at DESC")}
end
```  
Then in the questions controller, instantiate a answers variable to pass to the show view that calls the scope `ordered_by_creation`  
```ruby
  def show
    @answer = Answer.new
    @answers = @qustion.answers.ordered_by_creation
  end
```
