# Modal Sign In
[devise](https://github.com/plataformatec/devise) | [bootstrap](https://github.com/twbs/bootstrap-sass) | [simple form](https://github.com/plataformatec/simple_form) | [haml rails](https://github.com/indirect/haml-rails)

Create a new rails app with bootstrap and devise
```
rails new sign_me_in
cd sign_me_in
```
```
# Gemfile

gem 'devise'
gem 'bootstrap-sass', '~> 3.1.1'
gem 'simple_form'
gem 'haml-rails'
```
run `bundle`

Add to your assets pipeline 
```css
/* app/assets/stylesheets/bootstrap_and_css_overrides.css.scss */
@import 'bootstrap'
```
```javascript
// app/assets/javascripts/application.js

//= require bootstrap

```
Install simple form and devise
```
rails generate simple_form:install --bootstrap
rails generate devise:install
rails generate devise user
```
```erb
<!-- app/views/layouts/application_layout.html.erb -->
<% if notice %>
 <p class="notice"><%= notice %></p>
<% elsif alert %>
  <p class="alert"><%= alert %></p>
<% end %>

<%= render "/layouts/user_info" %>
```
Create a partial for the user info
```haml
/ app/views/layout/_user_info.html.haml

- if current_user
  = current_user.email
  = link_to t("buttons.campaign"), campaigns_path, class: 'btn btn-default'
  = link_to t("buttons.logout"), logout_path, class: "btn btn-default"
- else
  = render "/layouts/signin_modal"
  = link_to t("buttons.signin"), "#", class: "btn btn-default", data: { toggle: "modal", target: "#signin-modal" }
  = link_to t("buttons.signup"), signup_path, class: "btn btn-default"
```  
generate a sessions controller
```
rails generate controller sessions
```
```ruby
# app/controllers/sessions_controller.rb
class UsersController < ApplicationController

  def new
    @user = User.new
    @user.build_profile    # because it has_one
    # @user.questions.build (if has_many)
  end

  def create
    @user = User.new(user_params)
    if @user.save
      session[:user_id] = @user.id
      redirect_to root_path, notice: "Thank you for registering"
    else
      render :new
    end
  end

  private

  def user_params
    params.require(:user).permit(:email, :password, :password_confirmation,
                                  {profile_attributes: [:first_name, :last_name, :address, :id]})
  end
end
```
Add a create js to the sessions create view

```haml
/ app/views/sessions/create.js.haml

$('#signin-modal').modal('hide');
```

Create a layout partial for the [modal](http://getbootstrap.com/javascript/#modals)
```haml
/ app/views/layouts/_login_modal.html.haml

#signin-modal.modal.fade{"aria-hidden" => "true", "aria-labelledby" => "myModalLabel", role: "dialog", tabindex: "-1"}
  = simple_form_for "", url: login_path, html: {class: "form-horizontal"} do |f|
    .modal-dialog
      .modal-content
        .modal-header
          %button.close{"aria-hidden" => "true", "data-dismiss" => "modal", type: "button"} ×
          %h4#myModalLabel.modal-title Sign In
        .modal-body
          = flash_messages if flash[:alert]
          = f.input :email
          = f.input :password
          = f.submit "login", class: "btn btn-default"



        .modal-footer
          Don't have an account?
          = link_to "Sign up", signup_path, class: "btn btn-default"
```
Create a welcome index view and controller
```ruby
class WelcomeController < ApplicationController
  def index
    respond_to do |format|
      format.html { render :index }
      format.js   { render :index }
    end
  end
end
```