# Support Ticketing System  
[Tam's solution](https://github.com/tkbeili/march_cohort_quiz1_solution)  
Start by creating a new application  
```bash
rails new quiz_1_solution -d postgres

rails g resource support_request nam email department done:boolean message:text
```  
In the migration file make the default false for done.  
```ruby
class CreateSupportRequests < ActiveRecord::Migration
  def change
    create_table :support_requests do |t|
      t.string :name
      t.string :email
      t.string :department
      t.boolean :done, default: false
      t.text :message

      t.timestamps
    end
  end
end
```  

Or in the model you could have done this  
```ruby
class SupportRequest < ActiveRecord::Base

  after_initialize :set_defaults

  def set_default 
    self.done ||= false
  end
  
end
```  

You can add an index to the migration file  
```ruby
class CreateSupportRequests < ActiveRecord::Migration
  def change
    create_table :support_requests do |t|
      t.string :name
      t.string :email
      t.string :department
      t.boolean :done, default: false
      t.text :message

      t.timestamps
    end
    
    add_index :support_requests, :name
    add_index :support_requests, :email
    
    add
  end
end
```  

Add a default route for index  
```ruby

root "support_requests#index"

```  
and a controller method

```ruby
class SupportRequest < ActiveRecord::Base

  def index
    @support_requests = SupportRequest.all
  end
  
end
```  
Add a view file for it.
```ruby
# app/views/support_requests/index.html.erb
```

```ruby
gem 'quiet_assets'
gem 'bootstrap-sass'
```  

and a form for inputting requests on new  
```erb
# app/views/support_requests/new.html.erb
<%= form_for @support_requests do |f| %>

  <div class="form-group">
    <%= f.label :name %>
    <%= f.text_field :name, class: "form-control" %>
  </div>
  
  <div class="form-group">
    <%= f.label :email %>
    <= f.text_field :email, class: "form-control" %>
  </div>

  <div class="form-group">
    <%= f.submit class: "btn btn-default" %>
  </div>

<% end %>
```  
run a rake `db:create db:migrate` after you set up your database.yml  
  
Add some radio buttons to your form. Start with the model  
```ruby
# app/models/support_request.rb
class SupportRequest < ActiveRecord::Base

  before_initialize :set_default

  DEPARTMENT_SALES      = "sales".freeze
  DEPARTMENT_MARKETING      = "marketing".freeze
  DEPARTMENT_TECHNICAL      = "technical".freeze

  DEPARTMENTS = [ DEPARTMENT_SALES, 
      DEPARTMENT_MARKETING, 
      DEPARTMENT_TECHNICAL ]

  private

  def set_default 
    self.done |= false
  end
end
```  
Now we can add those classes as modules from the console like so `SupportRequest::DEPARTMENTS`. Let's fix up our form to use radio buttons  
```erb
# app/views/support_requests/new.html.erb
<%= form_for @support_request do |f| %>

  <div class="form-group">
    <%= f.label :name %>
    <%= f.text_field :name, class: "form-control" %>
  </div>
  
  <div class="form-group">
    <%= f.label :email %>
    <%= f.text_field :email, class: "form-control" %>
  </div>
  
  <% SupportRequest::DEPARTMENTS.each_with_index do |sr, i| %>
  
    <%= f.radio_button :department, sr, checked: sr == SupportRequest::DEPARTMENT_TECHNICAL %>
    <%= f.label :department, sr.capitalize %>
    
  <% end %>

  <div class="form-group">
    <%= f.label :message %>
    <%= f.text_area :message, class: "form-control" %>
  </div>

  <div class="form-group">
    <%= f.submit class: "btn btn-default" %>
  </div>

<% end %>
```  

Let's add validations
```ruby
class SupportRequestsController < ApplicationController

  def index
    @support_requests = SupportRequest.all
  end

  def new
    @support_request = SupportRequest.new
  end

  def create
      @support_request = Support.Request.
                new(support_request_params)
      if @suport_request.save
        redirect_to support_requests_path, notice: "Support Request Saved"
      else
        render :new
      end
  end

  private

  def support_request_params
    params.require(:support_request).
      permit([:name, :email, :message, :department])
  end


end

```  
Create an index
```erb
<h1>All Support Requestions</h1>

<table>
  <tr>
    <td>Name</td>
    <td>Email</td>
    <td>Department</td>
    <td>Message</td>
  </tr>
<% @support.requests.each do |sr| %>
    <tr>
      <td><%= sr.name.capitalize %> </tr>
      <td><%= sr.email %> </tr>
      <td><%= sr.department %> </tr>
      <td><%= sr.message %> </tr>
      <td><%= link_to "Edit", edit_support_request_path(sr), class: "btn btn-default" %></td>
      <td><%= link_to "Delete", sr, method: :delete, class: "btn btn-default" %></td>
      <td><%= link_to "Done or Undone", "javascript:void(0);", class: "btn btn-default" %></td>
    </tr>
  <% end %>
</table>
```  
Update your controller
```ruby
class SupportRequestsController < ApplicationController

  before_action :find_support_request, except: [:new, :create]
  def index
    @support_requests = SupportRequest.all
  end

  def new
    @support_request = SupportRequest.new
  end

  def create
      @support_request = Support.Request.
                new(support_request_params)
      if @suport_request.save
        redirect_to support_requests_path, notice: "Support Request Saved"
      else
        render :new
      end
  end

  def update
    if @support_request.update_attributes(support_request_path)
      redirect_to support_requests_path
    end
      render :edit
  end

  def destroy
    if @support_request.destroy
      redirect_to support_requests_path, notice: "Request deleted"
    else
      redirect_to support_requests_path, error: "Request denied"
    end
  end

  private

  def support_request_params
    params.require(:support_request).
      permit([:name, :email, :message, :department])
  end

  def find_support_requests
    @support_request = SupportRequests.find(params[:id])
  end

end
```  

```erb
application.hml.erb

<!DOCTYPE html>
<html>
<head>
  <title>Quiz1Solution</title>
  <%= stylesheet_link_tag    "application", media: "all", "data-turbolinks-track" => true %>
  <%= javascript_include_tag "application", "data-turbolinks-track" => true %>
  <%= csrf_meta_tags %>
</head>
<body class="container">

  <% if flash[:notice] %>
    <div class="alert alert-success">
      <%= flash[:notice] %>
    </div>
   <% elsif flash[:error] %>
  <div class="alert alert-danger">
      <%= flash[:error] %>
    </div>
   <% end %>
   
<%= yield %>

</body>
</html>

```  
## Faker Gem
Put `gem 'faker'` in your Gemfile.  

Then you can do something like this in rails c
```irb
20.times {
  SupportRequest.create(name: Faker::Name.name,
                      email: Faker::Internet.email,
                      department: SupportRequest::DEPARTMENTS.sample,
                      message: Faker::Lorem.sentence
                      )
}
```  
Add some pagination gems to your Gemfile  
```ruby
gem 'will_paginate'
gem 'will_paginate_bootstrap'
```  
Add pagination to your index file  
```erb
<h1>All Support Requestions</h1>

<table>
  <tr>
    <th>Name</th>
    <th>Email</th>
    <th>Department</th>
    <th>Message</th>
    <th></th>
    <th></th>
    <th></th>
  </tr>
<% @support.requests.each do |sr| %>
    <tr>
      <td><%= sr.name.capitalize %> </tr>
      <td><%= sr.email %> </tr>
      <td><%= sr.department %> </tr>
      <td><%= sr.message %> </tr>
      <td><%= link_to "Edit", edit_support_request_path(sr), class: "btn btn-default" %></td>
      <td><%= link_to "Delete", sr, method: :delete, class: "btn btn-default" %></td>
      <td><%= link_to "Done or Undone", "javascript:void(0);", class: "btn btn-default" %></td>
    </tr>
  <% end %>

  <tr>
    <td colspan="7" class="text-center">
    <%= will_paginate @support_requests, renderer: BootstrapPagination::Rails %>
  </tr>
</table>
```  
And update the index in your controller  
```ruby
class SupportRequestsController < ApplicationController

  before_action :find_support_request, only: [:edit, :update, :destroy]
  def index
    @support_requests = SupportRequest.order("done DESC").paginate({per_page: 5, page: params[:page]})
  end

  def new
    @support_request = SupportRequest.new
  end

  def create
    @support_request = Support.Request.
              new(support_request_params)
    if @suport_request.save
      redirect_to support_requests_path, notice: "Support Request Saved"
    else
      render :new
    end
  end

  def show
  end

  def update
    if @support_request.update_attributes(support_request_params)
      redirect_to support_requests_path
    end
      render :edit
  end

  def destroy
    if @support_request.destroy
      redirect_to support_requests_path, notice: "Request deleted"
    else
      redirect_to support_requests_path, error: "Request denied"
    end
  end

  private

  def support_request_params
    params.require(:support_request).
      permit([:name, :email, :message, :department])
  end

  def find_support_request
    @support_request = SupportRequest.find(params[:id])
  end

end
```  
Update your index for done and undaone:  
```erb
<h1>All Support Requestions</h1>

<table>
  <tr>
    <th>Name</th>
    <th>Email</th>
    <th>Department</th>
    <th>Message</th>
    <th></th>
    <th></th>
    <th></th>
  </tr>
<% @support_requests.each do |sr| %>
    <tr>
      <td><%= sr.name.capitalize %> </tr>
      <td><%= sr.email %> </tr>
      <td><%= sr.department %> </tr>
      <td><%= sr.message %> </tr>
      <td><%= link_to "Edit", edit_support_request_path(sr), class: "btn btn-default" %></td>
      <td><%= link_to "Delete", sr, method: :delete, class: "btn btn-default" %></td>


      <td>
        <% if sr.done? %>
          <%= button_to "Undone", support_request_path(sr, support_request: {done: false}), class: "btn btn-danger", method: :path %></td>
        <% else %>
          <%= button_to "Done", support_request_path(sr, support_request: {done: true}), class: "btn btn-default", method: :patch %>
        <% end %>
      </td>
    </tr>
  <% end %>

  <tr>
    <td colspan="7" class="text-center">
    <%= will_paginate @support_requests, renderer: BootstrapPagination::Rails %>
  </tr>
</table>
```  
Add a self method to search in your model  
```ruby
class SupportRequest < ActiveRecord::Base

  # before_initialize :set_default
  validates_presence_of :name, :email

  DEPARTMENT_SALES      = "sales".freeze
  DEPARTMENT_MARKETING      = "marketing".freeze
  DEPARTMENT_TECHNICAL      = "technical".freeze

  DEPARTMENTS = [ DEPARTMENT_SALES, 
                  DEPARTMENT_MARKETING, 
                  DEPARTMENT_TECHNICAL ]

  def self.search_by(search_term)
    where(["name LIKE :search_term OR email LIKE :search_term 
            OR message LIKE :search_term", 
            {search_term: search_term}]
          )
  end

  private

  def set_default 
    self.done ||= false
  end

end
```  
and add a method to search in your index controller  
```ruby
def index
  @support_requests = SupportRequest.order("done DESC").paginate({per_page: 5, page: params[:page]})
  if params[:search]
    @search_term = params[:search]
    @support_requests = @support_requests.search_by(@search_term)
  end
end
```  
Add to your index.html.erb  
```erb
<div class="row">
  <div class="col-md-6 col-sm-6">
    <h1>All Support Requestions</h1>
  </div>
  <div class="col-md-6 col-sm-6">
    <%= form_tag support_requests_path, method: :get do %>
      <%= text_field_tag :search, @search_term %>
      <%= submit_tag "Search", class: "btn btn-default" %>
    <% end %>
  </div>
</div>

<table>
  <tr>
    <th>Name</th>
    <th>Email</th>
    <th>Department</th>
    <th>Message</th>
    <th></th>
    <th></th>
    <th></th>
  </tr>
<% @support_requests.each do |sr| %>
    <tr>
      <td><%= sr.name.capitalize %> </tr>
      <td><%= sr.email %> </tr>
      <td><%= sr.department %> </tr>
      <td><%= sr.message %> </tr>
      <td><%= link_to "Edit", edit_support_request_path(sr), class: "btn btn-default" %></td>
      <td><%= link_to "Delete", sr, method: :delete, class: "btn btn-default" %></td>


      <td>
        <% if sr.done? %>
          <%= button_to "Undone", support_request_path(sr, support_request: {done: false}), class: "btn btn-danger", method: :path %></td>
        <% else %>
          <%= button_to "Done", support_request_path(sr, support_request: {done: true}), class: "btn btn-default", method: :patch %>
        <% end %>
      </td>
    </tr>
  <% end %>

  <tr>
    <td colspan="7" class="text-center">
    <%= will_paginate @support_requests, renderer: BootstrapPagination::Rails %>
  </tr>
</table>
```  
Update the model  
```ruby
def self.search_by(search_term)
    where(["LOWER(name) LIKE :search_term OR 
            LOWER(email) LIKE :search_term 
            OR LOWER(message) LIKE :search_term", 
            {search_term: "%#{search_term.downcase}%"}]
          )
  end
```  
You can use the rails helper method `mail_to` for the email link on your index page  
```erb
<td><%= mail_to sr.email %> </tr>
```  
Add session[:page] so you can stay on the page you are at when you set tickes to (un)done.  
```ruby
# app/controllers/support_requests_controller.rb

#...

def index

#...