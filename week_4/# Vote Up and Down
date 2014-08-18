# Vote Up and Down
  
Let's create both a model and controller with one command

```bash
rails g resource vote is_up:boolean question:references user:references
```  
Open the vote model and add validation for uniqueness based on user id  
```ruby
# app/models/vote.rb
class Vote < ActiveRecord::Base
  belongs_to :question
  belongs_to :user

  validates :question_id, uniqueness: {scope: :user_id}
end
```  
Add some route logic, and we can now remove the vote up and vote down methods from the questions controller    
```ruby
# config/routes.rb
AwesomeAnswers::Application.routes.draw do
 
  devise_for :users
  root "questions#index"
  get "/test" => "questions#test"
  resources :answers, only: [] do
    resources :comments
  end
  
  resources :questions do
    resources :votes, only: [:create, :update, :destroy]
    resources :answers
    post :search, on: :collection
  end
end
```
Add has many to the user model  
```ruby
class User < ActiveRecord::Base

#...

  has_many :votes, dependent: :destroy
  has_many :voted_questions, through: :votes, source: :question 

#...
  
end
```  
Add to has many relations to the user model for votes  
```ruby
# app/models/user.rb
class Question < ActiveRecord::Base

  #...
  
  has_many :categorizations, dependent: :destroy
  has_many :categories, through: :categorizations

  has_many :votes, dependent: :destroy
  has_many :voted_users, through: :votes, source: :user

  #...

end
```  
Instantiate a new vote in the questions controller and create a form for voting  
```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController

  #...
  
  def show
    @question = Question.find(params[:question_id] || params[:id])
    @answer = Answer.new
    @vote = Vote.new
    @answers = @question.answers.ordered_by_creation
  end
  
  #...
  
end
```
```haml
/ app/views/questions/show.html.haml

#...

    = form_for [@question, @vote] do |f|
      = f.hidden_field :is_up, value: true
      = f.submit "Vote Up", class: "btn btn-default"
    %br
    = form_for [@question, @vote] do |f|
      = f.hidden_field :is_up, value: false
      = f.submit "Vote down", class: "btn btn-default"
    %br

#...

```  
Your show page should now look like this  
```haml
= link_to "Edit", edit_question_path(@question), class: "btn btn-default"
%h1= @question.title
%p
  = @question.description
  %p
    = formatted_date(@question.created_at)
    = succeed "Vote" do
      %br
    Count: #{@question.vote_count}
    %br
    - if @question.categories.present?
      Categories (#{@question.categories.count}):
      = @question.categories.map(&:title).join(", ")
    %br
    
    = form_for [@question, @vote] do |f|
      = f.hidden_field :is_up, value: true
      = f.submit "Vote Up", class: "btn btn-default"
    %br
    = form_for [@question, @vote] do |f|
      = f.hidden_field :is_up, value: false
      = f.submit "Vote down", class: "btn btn-default"
    %br

    = button_to "Delete", @question, method: :delete, data: { confirm: "Are you sure you want to delete this question?" }, class: "btn btn-default"
    %br/
    = link_to "All Questions", questions_path

    %h2 Add an Answer
    - if @answer.errors.any?
      %ul
        - @answer.errors.full_messages.each do |message|
          %li= message

    = form_for @answer, url: question_answers_path(@question) do |f|
      = f.text_area :body, class: "form-control"
      %br
      = f.submit "Submit an answer", class: "btn btn-primary"

    %hr
    -@question.answers.each do |answer|
      = render "/answers/answer", answer: answer
```  
Add a create method to the votes controller with a before action, and vote params.    
```ruby
class VotesController < ApplicationController
  before_action :authenticate_user!

  def create
    @question   = Question.find.params(:question_id)
    @vote       = @question.votes.new(vote_params)
    @vote.user  = current_user
    if @vote.save
      redirect_to @question, notice: "Thanks for voting"
    else
      redirect_to @question, alert: "Your vote wasn't recorded!"
    end
  end

  private

  def vote_params
    params.require(:vote).permit(:is_up)
  end
end
```  
***note***: Change 'error' to 'alert' on your application layout and questions controller.  

Change the questions controller show method to find the current user, or instantiate a new vote    
```ruby  
class QuestionsController < ApplicationController
  #...
  
  def show
    @question = Question.find(params[:question_id] || params[:id])
    @answer   = Answer.new
    @vote     = current_user.vote_for(@question) || Vote.new
    @answers  = @question.answers.ordered_by_creation
  end
  
  #...

end
```  
Define a vote_for method in the user model  
```ruby
class User < ActiveRecord::Base

  #...
  
  def vote_for(question)
    Vote.where(question: question, user: self).first
  end
  
  #...
  
end

```  
Rails c fun  
```bash
current_user = User.find 1
@question = current_user.questions[0]
@vote = current_user.vote_for(@question) || Vote.new
@vote.persisted?
Vote.new.persisted?
Vote.first.persisted?
```  
Add an if else statement to the questions show page to show the option to 'undo' if voted, or to 'vote up' if not.   
```haml
    - if @vote.persisted? && @vote.is_up?
      = button_to "Undo", [@question, @vote], method: :delete, class: "btn btn-default"
    - else
      = form_for [@question, @vote] do |f|
        = f.hidden_field :is_up, value: true
        = f.submit "Vote Up", class: "btn btn-default"
```  
Add an update and destroy action to the votes controller, and a before action to find the question  
```ruby
class VotesController < ApplicationController
  before_action :find_question
  
  #...

  def destroy
    @vote = current_user.votes.find(params[:id])
    if @vote.destroy
      redirect_to @question, notice: "Vote removed"
    else
      redirect_to @question, alert: "Vote couldn't be removed"
    end
  end
  
  def update
    @vote = current_user.votes.find(params[:id])
    if @vote.update_attributes(vote_params)
      redirect_to @question, notice: "Vote updated."
    else
      redirect_to @question, alert: "Vote not updated"
    end
  end

  #...

  def find_question
    @question = Question.find(params[:question_id])
  end

end

```
