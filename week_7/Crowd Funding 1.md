# Crowd Funding 1: Users with Profiles  
```bash
rails new crowd_funding -T -d postgresql
```
**note**: we pass in `-T` to not install tests, and use `-d postgresql` to specify a postgres database.

Add the path to your database.yml to the gitignore file. For best practices, you can copy the default database.yml to database.yml.example for other developers to use.

You can then add some gems to your Gemfile  
```ruby
# Gemfile

# ...

gem 'thin'
gem 'haml-rails'
gem 'quiet_assets'
gem 'bootstrap-sass'
gem 'simple_form', github: 'plataformatec/simple_form'
gem 'bcrypt', '~> 3.1.7'

# ...
```
Run `bundle install` And add a bootstrap and css overrides file to your assets path.
```css
/* app/assets/stylesheets/bootstrap_and_css_overrides.css.scss */
@import 'bootstrap';
```
Let's create a user model, and install simple form.  
```bash
rails g resource user email password_digest
rails g simple_form:install --bootstrap
```
Read the output, and follow along.  
```bash
===============================================================================

  Be sure to have a copy of the Bootstrap stylesheet available on your
  application, you can get it on http://getbootstrap.com/.

  Inside your views, use the 'simple_form_for' with one of the Bootstrap form
  classes, '.form-horizontal' or '.form-inline', as the following:

    = simple_form_for(@user, html: { class: 'form-horizontal' }) do |form|

===============================================================================
```
Open up the migration file for the user model and add an index  
```ruby
# db/migrations/1237129837123217_create_users.rb
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :email
      t.string :password_digest

      t.timestamps
    end

    add_index :users, :email

  end
end
```
Let's make a controller called welcome.
```bash
rails g controller welcome index
```
Add a root to the routes.  
```ruby
# congfig/routes.rb

# ...

  root "welcome#index"
  
# ...
```
Open up the application layout and wrap the yield code with a div of class container.  
```erb

  <div class="container">
    <%= yield %>
  </div>
  
```
Add a get signup route to your routes
```ruby
Rails.application.routes.draw do
  get "signup" => "users#new", as: :signup

  resources :users

  root "welcome#index"
end
```
Then on our welcome back, we can add a link button to signup.
```erb
= link_to "sign up", signup_path, class: "btn btn-default"
```

Add to the user model for encryption and decryption
```ruby
# app/models/user.rb

class User < ActiveRecord::Base
  has_secure_password
end
```
Add a form to the user new view
```haml
/ app/views/users/new.html.haml

= simple_form_for @user, html: {class: "form-horizontal"} do |f|
  = f.input :email
  = f.input :password
  = f.input :password_confirmation
  = f.submit "sign up", class: "btn btn-default"
```
Change the user model to validate email
```ruby
# app/models/user.rb

class User < ActiveRecord::Base
  has_secure_password

  validates :email, presence: true, uniqueness: true, email_format: true
end
```
Add an email validator class. First create a folder in your app directory called validators.
```ruby
# app/validators/email_format_validator.rb

class EmailFormatValidator < ActiveModel::EachValidator

  EMAIL_REGEX = /^(|(([A-Za-z0-9]+_+)|([A-Za-z0-9]+\-+)|([A-Za-z0-9]+\.+)|([A-Za-z0-9]+\++))*[A-Za-z0-9]+@((\w+\-+)|(\w+\.))*\w{1,63}\.[a-zA-Z]{2,6})$/i

  def validate_each(object, attribute, value)
    unless value =~ EMAIL_REGEX
      object.errors[attribute] << (options[:message] || "is not formated correctly")
    end
  end

end

```

```RUBY

def create
  @user = User.new(user_params)
  if @user.save
    redirect_to root_path, notice: "Thank you for registering"
  else
    render :new
  end
end

private

def user_params
  params.require(:user).permit(:email, :password, :password_confirmation)
end
```
If you're having an issue with something, try uncommenting the line in your config application.rb around line 27
```ruby
# config/application.rb

# ...

    config.autoload_paths += Dir[Rails.root.join('my', 'locales', '*.{rb,yml}').to_s]
    
# ...
```
To authenticate the user useing a session, in the users controller, add this to the application controller

```ruby
class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception

  private

  def current_user
    @current_user ||= User.find(session[:user_id]) if session[:user_id]
  end

  helper_method :current_user

  def authenticate_user!
    redirect_to root_path, alert: "You need to sign in" if current_user.nil?
  end

end
```
And add a session to the user controller
```ruby
# app/controllers/users_controller.rb

# ...

  def create
    @user = User.new(user_params)
    if @user.save
      session[:user_id] = @user.id
      redirect_to root_path, notice: "Thank you for registering"
    else
      render :new
    end
  end
  
# ...
```
We need to generate a model for the profile.  
```
rails generate model profile first_name last_name address user:references
rake db:migrate
```
Then open up the User model, and add a relation.  
```ruby
# app/models/user.rb

# ...

  has_one :profile
  accepts_nested_attribues_for :profile

# ...
```
Now we can add some fields for a profile form to the new user form
```haml
/ app/views/users/new.html.haml

= simple_form_for @user, html: {class: "form-horizontal"} do |f|
  = f.input :email
  = f.input :password
  = f.input :password_confirmation

  = f.fields_for :profile do |p|
    = p.input :first_name
    = p.input :last_name
    = p.input :address

  = f.submit "sign up", class: "btn btn-default"
```
Now add to the users controller to build a profile.  
```ruby
# app/controllers/users_controller.rb

# ...

   def new
    @user = User.new
    @user.build_profile    # because it has_one
    # @user.questions.build (if has_many)
  end

# ...
```
Add the parameters to user params in the users controller.
```ruby

# ...

  def user_params
    params.require(:user).permit(:email, :password, :password_confirmation, 
                                  {profile_attributes: [:first_name, :last_name, :address, :id]})
  end

# ...

```
To add the flash messages to our application layout, there's a great way we can do this.
```erb
<!- app/views/layouts/application.html.erb

  <div class="container">
    <%= flash_messages %>
    <%= yield %>
  </div>


```

```ruby
# app/helpers/application_helper.rb

module ApplicationHelper

  def flash_messages
    flashes = ""
    flash.each do |type, value|
      flashes += content_tag(:div, value, class: flash_class(type.to_sym))
    end
    content_tag(:div, flashes.html_safe)
  end

  def flash_class(type)
    case type
    when :notice then "alert altert-info"
    when :success then "alert alert-success"
    when :alert then "alert alert-danger"
    end
  end
end
```
Add some nice routes to your routes
```ruby
# config/routes.rb

Rails.application.routes.draw do
  get "signup" => "users#new", as: :signup
  get "login" => "sessions#new", as: :login
  post "/login" => "sessions#create"
  delete "logout" => "sessions#destroy", as: :logout

  resources :users

  root "welcome#index"
end
```
THen create a SessionsController `rails g controller sessions`
```ruby
# app/controllers/sessions_controller.rb

class SessionsController < ApplicationController

  def new

  end

  def create
    user = User.find_by_email(params[:email])
    if user && user.authenticate(params[:password])
      session[:user_id] = user.id
      redirect_to root_path, notice: "Thanks for logging in!"
    else
      flash.now.alert = "Email or password is invalid"
      render :new
    end
  end

  def destroy
    session[:user_id] = nil
    redirect_to root_path, notice: "You've been logged out"
  end
  
end
```
And add a new sessions view
```haml
/ app/views/sessions/new.html.haml

%h1 Sign In

= simple_form_for "", url: login_path, html: {class: "form-horizontal"} do |f|
  = f.input :email
  = f.input :password
  = f.submit "login", class: "btn btn-default"

%br

%p
  Don't have an account?
  = link_to "Sign up", signup_path
```
Modify the welcome view
```haml
/ app/views/welcome/index.html.haml

%h1 Welcome

- if current_user
  = current_user.full_name
  = link_to "logout", logout_path, class: "btn btn-default", method: :delete
- else
  = link_to "Sign in", login_path, class: "btn btn-default"
  = link_to "Sign up", signup_path, class: "btn btn-default"
```
Make sure to add a full name method to the profile model.
```ruby
# app/models/profile.rb

class Profile < ActiveRecord::Base
  belongs_to :user

  def full_name
    if first_name || last_name
      "#{first_name} #{last_name}".squeeze(" ").strip
    end
  end
end
```
And you can add to your user model a delegate. `delegate :full_name, to: :profile`  
```ruby
# app/models/user.rb

class User < ActiveRecord::Base
  has_secure_password

  validates :email, presence: true, uniqueness: true, email_format: true
  has_one :profile

  accepts_nested_attributes_for :profile, allow_destroy: true

  delegate :full_name, to: :profile

end
```
  
P.S. there's something called the [law of demeter](http://en.wikipedia.org/wiki/Law_of_Demeter), which basically says "You're a newspaper boy. You go and deliver the newspaper to a person, and the person gives you money. You don't care how the person gives the money. You don't say 'pick up your wallet, open it, take out $5.00, and give it to me.' That doesn't matter. You just say, 'Here's the paper, give me money.'" The idea here is to have loose coupling, rather than try to guess too much about what the other class is doing.  
  
The idea with object oriented design is like if, hmm a lot of people blame Rails for being kind of too easy in this case. Let's say you have an object User, and an object Profile, and let's say we have a view, you shouldn't be jumping by one to get to the other. Maybe the view should just ask the user for the full name, and the user knows to get this from the profile.  
  
The way to do this in Rails, because it's fairly common is to use `delegate`, like in our case `delegate :full_name, to: :profile, prefix: true` If we do prefix true, we have to put the actual model name first, like `profile_full_name` rather than just `full_name`.  
  
## Set Locale
Add a file called es.yaml to your config locales directory.  
```yaml
# config/locales/es.yaml

es:
  hello: "Hola"
```
Then add a method to your application controller  
```ruby
# app/controllers/application_controller.rb

# ...


  def set_locale heh
    I18n.local = params[:locale] if params[:locale].present?
  end
  
# ...
```