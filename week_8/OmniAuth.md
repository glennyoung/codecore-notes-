# OmniAuth  
| [OmniAuth](https://github.com/intridea/omniauth) |  
OmniAuth integrates nicely with Devise. So, we're going to use this with Awesome Answers today.  
  
Google shares the email of the user with you, while Twitter doesn't. Facebook does something that others don't, etc. The main difference is that different services do things a little difference. However, we can authenticate a user from a variety of sources using OpenAuth.  
  
Add omniauth for twitter to your gemfile
```ruby
# Gemfile

# ...

gem 'omniauth-twitter'

# ...

```
Then go to [dev.twitter.com](https://dev.twitter.com/), sign in, click on your picture on the upper right, select 'my applications', and 'create application'. Fill out the form, (use (http://lvh.me:4000) for the url, and go to settings, then check ***allow this application to be signed in with twitter***  
  
Now, we can use our API keys. Go to config > initializers and make an api keys for twitter file and add it to your git ignore
```ruby
# .gitignore

/config/initializers/api_keys_for_twitter.rb

```
```ruby
# /config/initializers/api_keys_for_twitter.rb

ENV["TWITTER_API_KEY"] = "[Your-API-Key"]
ENV["TWITTER_API_SECRET"] = ["Your-API-Secret"]
```
Now, go to devise.rb and add this line
```ruby
# config/initializers/devise.rb

# ...

  config.omniauth :twitter, ENV["TWITTER_API_KEY"], ENV["TWITTER_API_SECRET"]

# ...
```
Add 'omniauthable' to the devise section of the user model
```ruby
# app/models/user.rb

  devise :database_authenticatable, :registerable, :omniauthable,
         :recoverable, :rememberable, :trackable, :validatable

# ...
```
Generate a new migration  
```
rails g migration make_user_email_optional
```
Then open up the migration file and add some candy  
```ruby
class MakeUserEmailOptional < ActiveRecord::Migration
  def change
    change_column :users, :email, :string, null: true
    remove_index :users, :email
    add_index :users, :email
  end
end
```
Generate a new controller.
```
rails generate controller users/omniauth_callbacks
```
Now that we've generated the controller, we need to set up the routes. We want to tell devise about the callbacks. 
```ruby
# routes.rb
 # ...

  devise_for :users, controllers: { omniauth_callbacks: "users/omniauth_callbacks" }

  # ...

```
Now, in your controllers > users > omniauth callbacks controller, change it to inherit from Devise, and add a twitter method.
```ruby
# app/controllers/users/omniauth_callbacks_controller.rb.   

class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController

  def twitter
    oauth_data = request.env["omniauth.auth"].to_hash

    user = User.find_or_create_from_twitter(oauth_data)

    sign_in user

    redirect_to questions_path

    # render json: request.env['omniauth.auth'].to_json
  end

end
```
```haml
/ app/views/layouts/application.html.haml

= link_to "Sign In WIth Twitter", user_omniauth_authorize_path(:twitter), class: "btn btn-default"

```
```ruby
# app/models/user.rb

  def self.find_or_create_from_twitter(oauth_data)
    user = User.where(provider: :twitter, uid: oauth_data[:uid]).first
    
    unless user
      name = oauth_data["info"]["name"].split(" ")
      user = User.create!(first_name: name[0], 
                          last_name: name[1],
                          password: Devise.friendly_token[0, 20],
                          provider: :twitter,
                          uid: oauth_data["uid"])
    end
    user
  end

  def email_required?
    false
  end
```

Generate a new migration.
```
rails g migration add_omniauth_fields_to_users provider uid
```
Then in your omniauth controller, you can just add another method for facebook, google, etc. And add to your config/initializers/devise.rb.
## Play with the Twitter Gem in irb  
  
`gem istall twitter`  
  
[twitter gem](https://github.com/sferik/twitter)  
  

You can hard code these for now, because we are just playing around with it in IRB
```ruby

client = Twitter::REST::Client.new do |config|
  config.consumer_key        = "YOUR_CONSUMER_KEY"
  config.consumer_secret     = "YOUR_CONSUMER_SECRET"
  config.access_token        = "YOUR_ACCESS_TOKEN"
  config.access_token_secret = "YOUR_ACCESS_SECRET"
end

```
We can just copy/paste this stuff from our callback, if we want. Or access them from the console.