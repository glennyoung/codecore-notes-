# Active Admin  

Add the gem [Active Admin](https://github.com/gregbell/active_admin) to your Gemifle
```Ruby
# Gemfile

# ...

gem 'activeadmin', github: 'gregbell/active_admin'

# ...
```
Then in terminal
```bash
bundle install
rails generate active_admin:install
rails generate active_admin:resource question
rake db:migrate
rails server
```
Then to login to the [admin panel](http://localhost:3000/admin/login), username: admin@example.com, password: password. 
  
Now, the will paginate gem will not work with active admin however the [kapinari](https://github.com/amatsuda/kaminari) gem will!  
  
Using the active admin panel. You can perform tasks on the tabs for the models you have generated. Let's generate an active admin resource for answers.  
```bash
rails g active_admin:resource answer
```
Let's add [can can](https://github.com/ryanb/cancan)
```ruby
# Gemfile

# ...

  gem 'cancan'

# ...
```
Then install it
```bash
bundle install
rails g cancan:ability
```
Then open up the class ability
```ruby
# app/models/ability.rb

class Ability
  include CanCan::Ability

  def initialize(user)
    user ||= User.new

    can :manage, Question, user_id: user.id

    can :manage, Answer, user_id: user.id
        user.id = answer.id || user.id == answer.question.id
    end

  end
end
```
Modify the show page to show edit delete if user has permissions.   
```ruby
# app/views/questions/show.html.haml

- if can? :edit, @question
  = button_to "Edit", edit_question_path(@question)
- if can? :destroy, @question
  = button_to "Delete", @question, method: :delete, data: { confirm: "Are you sure you want to delete this question?" }, class: "btn btn-default"
  
```

And then change the edit action of the questions controller
```ruby
# app/controllers/questions_controller.rb

# ...

  def edit
    redirect_to root_path, alert: "Access Denied" unless can? :edit, @question
  end
  
# ...
```

Create a migration to add an 'is admin' boolean field to the user model  
```ruby
# rails g migration add_is_admin_to_users

class AddIsAdminToUsers < ActiveRecord::Migration
  def change
    add_column :users, :is_admin, :boolean, default: false
  end
end
```
Add to the ability model  
```ruby
# app/models/ability.rb

class Ability
  include CanCan::Ability

  def initialize(user)
    user ||= User.new

    if user.is_admin?
      can :manage, :all
    end

    can :manage, Question, user_id: user.id

    can :manage, Answer, user_id: user.id
        user.id = answer.id || user.id == answer.question.id
    end

  end
end
```
Then run `rake db:migrate`
