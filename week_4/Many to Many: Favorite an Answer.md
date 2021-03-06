#Many to Many: Favorite an Answer  
  
Generate a favorite model that has question and user references
```bash
rails g resource favorite question:references user:references
```  
Resources
```ruby
# config/routes.rb
resources :questions do
  resources :favorites, only: [:create, :destroy]
  resources :votes, only: [:create, :update, :destroy]
  resources :answers
  post :search, on: :collection
end
```  
Add create and destroy methods to the favorites controller with a before action that authenticates the user.  
```ruby
# app/controllers/favorites_controller.rb

class FavoritesController < ApplicationController
  before_action :authenticate_user!

  def create
    @question = Question.find params[:question_id]
    @favorite = @question.favorites.new
    @favorite.user = current_user

    if @favorite.save
      redirect_to @question, notice: "Thank you for favoriting"
    else
      redirect_to @question, alert: "You question could not be saved"
    end
  end

  def destroy
  end
end
```  
How can we do the favorite on the show page? We can get rid of the vote count.    
```haml
/ app/views/questions/show.html.haml

 = button_to "Favorite", question_favorites_path(@question), method: :post
 
```  
Add has many relations to the question model
```ruby

#...

# app/models/question.rb
  has_many :favorites, dependent: :destroy
  has_many :favorited_users, through: :favorites, source: :user
  
#...

```
And the user model
```ruby
# app/models/user.rb

#...

has_many :favorites, dependent: :destroy
has_many :favorited_questions, through: :favorites, source: :question

#...
```  
How do we enforce at the model level that the user doesn't vote twice?  
```ruby
# app/models/favorite.rb

#...

validates :user_id, uniqueness: {scope: :question_id}

#...
```  
## Add an 'unfavorite' button
```haml
/ app/questions/views/show.html.haml

/...
%p= button_to "Unfavorite", question_favorites_path(@question, @favorite), method: :delete, class: "btn btn-default"
/...
```  
In the questions controller, add an instance variable for @favorite in the show method.  
```ruby

#...
# app/controllers/questions_controller.rb

  def show
    @question = Question.find(params[:question_id] || params[:id])
    @answer   = Answer.new
    @favorite = current_user.favorite_for(@question)
    @vote     = current_user.vote_for(@question) || Vote.new
    @answers  = @question.answers.ordered_by_creation
  end

#...
```  
Add a `favorite_for` method in the User model.  
```ruby
# app/models/user.rb

#...
  def favorite_for(question)
    favorites.where(question: question).first
  end
  
#...
```  
Add to the destroy method in the favorites controller, and add a before action to find a question.  
```ruby
# app/controllers/favorites_controller.rb

class FavoritesController < ApplicationController
  before_action :authenticate_user!
  before_action :find_question

  def create
    @question = Question.find params[:question_id]
    @favorite = @question.favorites.new
    @favorite.user = current_user

    if @favorite.save
      redirect_to @question, notice: "Thank you for favoriting"
    else
      redirect_to @question, alert: "Your question could not be saved"
    end
  end

  def destroy
    @favorite = current_user.favorites.find(params[:id])
    if @favorite.destroy
      redirect_to @question, "You have unfavorited"
    else
      redirect_to @question, alert: "Couldn't unfavorite"
    end
  end

  private

  def find_question
    @question = Question.find params[:question_id]
  end

end


```  
Change the questions show view to display 
```haml
/ app/views/questions/show.html.haml


/...

- if @favorite
  %p= button_to "Unfavorite", question_favorites_path(@question, @favorite), method: :delete, class: "btn btn-default"
- else
  %p= button_to "Favorite", question_favorites_path(@question), method: :post, class: "btn btn-default"
  
/...

```  
Add a list of favorited users to the show page.   
```haml
/ app/views/quetions/show.html.haml

# ...

- if @question.favorited_users.present?
  Favoritd Users:
  = @question.favorited_users.map(&:full_name).join(", ")
  
# ...
```
