# User Authentication with Devise  
[Devise](https://github.com/plataformatec/devise) | [Devise w/ OAUTH](https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview) |  
  
Add `gem 'devise'` to your gemfile  
```ruby
# Gemfile
source 'https://rubygems.org'

gem 'rails', '4.0.2'
gem 'pg'
gem 'thin'
gem 'bootstrap-sass', '~> 3.1.1.0'
gem 'haml-rails'
gem 'devise'

gem 'sass-rails', '~> 4.0.0'
gem 'uglifier', '>= 1.3.0'
gem 'coffee-rails', '~> 4.0.0'
gem 'jquery-rails'
gem 'turbolinks'
gem 'jbuilder', '~> 1.2'

group :development do
  gem 'hirb'
  gem 'interactive_editor'
  gem 'awesome_print'
  gem 'quiet_assets'
  gem 'better_errors'
  gem 'binding_of_caller'
end

group :doc do
  gem 'sdoc', require: false
end
```  
Then run (in terminal):  
```bash
bundle install
rails generate devise:install
```  
Then, follow the instructions (we can skip step 4, since we're using Rails 4.1):  
```bash
Some setup you must do manually if you haven\'t yet:

  1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { :host => 'localhost:3000' }

     In production, :host should be set to the actual host of your application.

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root :to => "home#index"

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

  4. If you are deploying on Heroku with Rails 3.2 only, you may want to set:

       config.assets.initialize_on_precompile = false

     On config/application.rb forcing your application to not access the DB
     or load models when precompiling your assets.

  5. You can copy Devise views (for customization) to your app by running:

       rails g devise:views
```  
If you want a model to be linked with devise, use `rails generate devise [model-name]` so in this case, we'll try:  
`rails generate devise user first_name:string last_name:string`  
Then we can run `rake db:migrate` or `bundle exec rake db:migrate`.  
  
Open up the User model  
```ruby
# app/models/user.rb  
class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable
end
```  
You can now check our [routes](http://localhost:3000/rails/info/routes) and see all the new paths for users.  
  
Let's set up some authentications in the questions controller  
```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController
  before_action :authenticate_user!, except: [:index, :show]
# more code...
```  
You can alter the form to look differently, if you'd like under new sessions  
```erb
# app/views/devise/sessions/new.html.erb 
<h2>Sign in</h2>

<%= form_for(resource, :as => resource_name, :url => session_path(resource_name)) do |f| %>
  <div><%= f.label :email %><br />
  <%= f.email_field :email, :autofocus => true, class: "form-control" %></div>

  <div><%= f.label :password %><br />
  <%= f.password_field :password, class: "form-control" %></div>

  <% if devise_mapping.rememberable? -%>
    <div><%= f.check_box :remember_me, class: "form-control" %> <%= f.label :remember_me %></div>
  <% end -%>

  <div><%= f.submit "Sign in", class: "btn btn-default" %></div>
<% end %>

<%= render "devise/shared/links" %>
```  
When we hit 'sign up' notice the default fields for Devise are to use use email, password, and password confirmation.  
  
Add a sign out button to the application layout if user is signed in, and sign in if not signed in.    
```haml
/ app/views/application/application.layout.haml
!!!
%html
  %head
    %title AwesomeAnswers
    = stylesheet_link_tag    "application", media: "all", "data-turbolinks-track" => true
    = javascript_include_tag "application", "data-turbolinks-track" => true
    = csrf_meta_tags
  %body
    .container
      %nav.nav-main
        - # link_to "About Us", about_us_path, class: "btn btn-default nav-btn", id: "about"
        - # link_to "FAQ", faq_path, class: "btn btn-default nav-btn"
        - # link_to "Help", help_path, class: "btn btn-default nav-btn"
        = link_to "home", questions_path, class: "btn btn-default"
        = link_to "New Question", new_question_path, class: "btn btn-default nav-btn"
        .pull-right
          - if user_signed_in?
            Hello
            = current_user.full_name
            = link_to "sign out", destroy_user_session_path, method: :delete, class: "btn btn-default"
          - else
            = link_to "sign in", new_user_session_path, class: "btn btn-default"
      - if flash[:notice] || flash[:error]
        %h3= flash[:notice] || flash[:error]

      =yield
```  
Add first and last name fields to devise new user registration form  
```erb
<h2>Sign up</h2>

<%= form_for(resource, :as => resource_name, :url => registration_path(resource_name)) do |f| %>
  <%= devise_error_messages! %>
  
  <div><%= f.label :first_name %><br />
  <%= f.text_field :first_name, class: "form-control" %></div>

  <div><%= f.label :last_name %><br />
  <%= f.text_field :last_name, class: "form-control" %></div>

  <div><%= f.label :email %><br />
  <%= f.email_field :email, :autofocus => true, class: "form-control" %></div>

  <div><%= f.label :password %><br />
  <%= f.password_field :password, class: "form-control" %></div>

  <div><%= f.label :password_confirmation %><br />
  <%= f.password_field :password_confirmation, class: "form-control" %></div>

  <div><%= f.submit "Sign up", class: "btn btn-default" %></div>
<% end %>

<%= render "devise/shared/links" %>
```  
To allow these fields to the strong parameters, the problem is we don't have access to the devise controller, because it's embeded in the gem. The cool thing is we can do this in the application controller.  
```ruby  
# app/controllers/applications_controller.rb
class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception

  before_action :configure_devise_params, if: :devise_controller?

  private
  
  def configure_devise_params
    devise_parameter_sanitizer.for(:sign_up) << 
                                    [:first_name, :last_name]
    devise_parameter_sanitizer.for(:account_update) << 
                                    [:first_name, :last_name]
  end

end
```  
add a full name method to the User model
```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable


  def full_name
    if first_name || last_name
      "#{first_name} #{last_name}".squeeze.strip
    else
      email
    end
  end
  
end
```  
How do we made a 'user has many questions' or 'user has many answers' relationship?  
```bash
rails generate migration add_user_references
```
Then open up the migration file and add user references to questions and answers    
```ruby
#migration/asdkjaslkdj/add_user_references.rb
class AddUserReferences < ActiveRecord::Migration
  def change
    add_reference :questions, :user, index: true
    add_reference :answers, :user, index: true
  end
end
```  
Add 'has many' relation to the user model  
```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable

  has_many :questions
  has_many :answers


  def full_name
    if first_name || last_name
      "#{first_name} #{last_name}".squeeze.strip
    else
      email
    end
  end
  
end
```  
Add belongs to user to the question model  
```ruby
# app/models/user.rb
# ...
belongs_to :user
#...
```  
Add a current user to the create action in the questions controller
```ruby
# app/controllers/questions_controller.rb
# this:
# @question.user = current_user
# or:
@question = current_user.questions.new(question_attributes)
```  
We can change the private method find_questions in the questions controller to find based on the current user, or redirect back to the root path with an alert.    
```ruby
  def find_question
    #@question = Question.find(params[:question_id] || params[:id])
    # @question = current_user.questions.find(params[:question_id] || params[:id])
    @question = current_user.questions.find_by_id(params[:id])
    redirect_to root_path, alert: "Access Denied" unless @question
  end
```  
***note***: We can remove show from the before action to find question, and modify the show method to find its own question, if we wish for unregistered (or non signed in) users to be able to view questions.  
```ruby
# app/controllers/questions_controller.rb
#...
before_action :find_question, 
                  only: [:edit, :destroy, :update, :vote_up, :vote_down]

#...
def show
  @question = Question.find(params[:question_id] || params[:id])
  @answer = Answer.new
  @answers = @question.answers.ordered_by_creation
end
# ...
```  
If you have old questions without user ids, you can add them like by running this in your rails console  
```bash
Question.update_all(user_id: 1)
```  
Inside the answer's controller, I can do the same thing in the create method. And modify the destroy method to make sure the user is the current user.  
```ruby
# app/controllers/answers_controller.rb
#...

  def create
    #@question = Question.find params[:question_id]
    @answer = @question.answers.new(answer_attributes)
    @answer.user = current_user
    if @answer.save
      redirect_to @question, notice: "Answer created successfully."
    else
      render "/questions/show"
    end
  end
#...

  def destroy
    @answer = @question.answers.find(params[:id])
    if @answer.user = current_user && @answer.destroy
      redirect_to @question, notice: "Answer deleted"
    else
      redirect_to @question, error: "We had trouble deleting the answer"
    end
  end

#...

```
