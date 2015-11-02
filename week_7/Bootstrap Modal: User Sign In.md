# Bootstrap
## Modal: User Sign In


Makesure you have bootstrap.js included in your application.js
```javascript
//= require jquery
//= require jquery_ujs
//= require bootstrap
//= require turbolinks
//= require_tree .
```
Create a partial for the modal in your layouts
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
Update your sessions controller create action with a resond to
```ruby
# app/controllers/sessions_controller.rb

# ...

  def create
    user = User.find_by_email(params[:email])
    respond_to do |format|
      if user && user.authenticate(params[:password])
        session[:user_id] = user.id
        format.html {redirect_to root_path, notice: "Thanks for logging in!"}
        format.js  do
          flash.now.notice = "Thanks for signing in."
          render
        end
      else
        flash.now.alert = "Email or password is invalid"
        format.html { render :new }
        format.js   { render :new }
      end
    end
  end

# ...

```

Add a couple js haml views for new and create sessions  
```haml
/ app/views/sessions/create.js.haml

$('#signin-modal').modal('hide');
$('#flash-messages').html('');
$('#flash-messages').hide('');
$('#flash-messages').html("#{j flash_messages}");
$('#flash-messages').fadeIn(5000);
setTimeout(function() { $("#flash-messages").fadeOut(1500) }, 4000);

$('#user-info').html("#{j render '/layouts/user_info'}");
```
```haml
/ app/views/sessions/new.js.haml

$("#signin-modal").replaceWith("#{j render '/layouts/signin_modal'}");
```
## WYSIWYG: CRUD

```
rails g resource campaign title details:text target:integer end_date:datetime user:references
```
Add some methods to the campaigns controller  
```ruby
# app/controllers/campaigns_controller.rb

class CampaignsController < ApplicationController
  def new
    @campaign = Campaign.new
  end

  def create
    @campaign = Campaign.new(campaign_params)
    if @campaign.save
      redirect_to root_path, notice: "success"
    else
      render :new
    end
  end

  private

  def campaign_params
    params.require(:campaign).permit(:title, :details, :target, :end_date)
  end
end
```
Add `has_many :campaigns` to the user model  
Add a campaign view
```haml
/ app/views/campaings/new.html.haml

= simple_form_for @campaign, html: { class: "form-horizontal" } do |f|
  = f.input :title
  = f.input :details
  = f.input :target
  = f.input :end_date
  = f.submit "Submit", class: "btn btn-default"
```
Add a link to new create campaigns to the index page
```haml
/ app/views/welcome/index.html.haml

%h1= t "hello"
%p Find me in app/views/welcome/index.html.haml
= link_to "New Campaign", new_campaign_path, class: "btn btn-default"
```
Update the welcome controller index action to include all the campaigns
```ruby
# app/controllers/welcome_controller.rb

class WelcomeController < ApplicationController
  def index
    @campaigns = Campaign.all
  end
end

```
And on the welcome index view, to display each campaign in a well.  
```haml
/ app/views/welcome/index.html.haml

%h1= t "hello"
%p Find me in app/views/welcome/index.html.haml
= link_to "New Campaign", new_campaign_path, class: "btn btn-default"

- @campaigns.each do |c|
  .well
    %h4= c.title
    %p= c.details.truncate(50)
    =link_to "View", c, class: "btn btn-default", data: {toggle: "modal", target: "#campaign" }, remote: true

```

Modify the show action in the campaigns controller
```ruby
# app/controllers/campaigns_controller.rb

# ...

  def show
    @campaign = Campaign.find(params[:id])
    respond_to do |format|
      format.html { render :show }
      format.js { render :show }
    end
  end

#...
```
Create a partial for your campaign modal in the layouts
```haml
/ app/views/layouts/_campaign_modal.html.haml

.modal-dialog
  .modal-content
    .modal-header
      %button.close{"aria-hidden" => "true", "data-dismiss" => "modal", :type => "button"} &times;
      = @campaign.title
      .modal-body
        %p
          Details: #{@campaign.details}
          Target: #{@campaign.target}
```
Add a show javascript file for the campaigns views
```haml
/ app/views/campaigns/show.js.haml

$("#campaign").html("#{j render '/layouts/campaign_modal'}");
```
Create a div on the application layout view to actually show to campaign modal
```haml
      #campaign.modal.fade{"aria-hidden" => "true", "aria-labelledby" => "myModalLabel", role: "dialog", tabindex: "-1"}
```
## Pagination  

Add the gem `gem 'will_paginate-bootstrap'` to your Gemfile.

Add this line to your welcome index
```haml


= will_paginate @campaigns, renderer: BootstrapPagination::Rails
```

Add paginate method to your welcome controller index action
```ruby

class WelcomeController < ApplicationController
  def index
    @campaigns = Campaign.paginate(params[:campaign]).order('created_at DESC')
  end
end
```
To the campaigns javascript file add the following
```coffeescript
# app/assets/javascripts/campaigns.js.coffee

$ ->
  $(document).on "click", ".pagination a", ->
    $(".pagination a").css("opacity", 0.25)
    $.getScript($(@).attr("href"))
    false

```
Modify the welcome index
```haml
/ app/views/welcome/index.html.haml

%h1= t "hello"
%p Find me in app/views/welcome/index.html.haml
= link_to "New Campaign", new_campaign_path, class: "btn btn-default"

#campaigns= render "campaigns_with_pagination"
```
Add a campaigns with pagination partial to the welcome views folder
```haml
/ app/views/welcome/_campaigns_with_pagination.html.haml

- @campaigns.each do |c|
  .well
    %h4= c.title
    %p= c.details.truncate(50)
    %p
      =link_to "View", c, class: "btn btn-default", data: {toggle: "modal", target: "#campaign" }, remote: true

= will_paginate @campaigns, renderer: BootstrapPagination::Rails
```
Add an index js haml
```haml
/ app/views/welcome/index.js.haml

$("#campaigns").html("#{j render 'campaigns_with_pagination'}");
$("html, body").animate({scrollTop:0}, 2000);
```
Modify the welcome contoller index action to respond to html or javascript
```ruby
# app/controllers/weclome_controller.rb

class WelcomeController < ApplicationController
  def index
    @campaigns = Campaign.paginate(page: params[:page], per_page: 20).order("created_at DESC")
    respond_to do |format|
      format.html { render :index }
      format.js   { render :index }
    end
  end
end
```
