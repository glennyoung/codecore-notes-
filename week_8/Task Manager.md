# Task Manager        
***Warning***: These notes are incomplete and are meant as a __reference only__. Use at your own risk!  
  
Let's get started with a new task manager. We won't have any user login,
just projects, and tasks.  
  
  * Projects have titles and descriptions, and
have many tasks.
  * Tasks can be either complete or incomplete and should be marked by
    ajax.
  * Everything should be Test Driven  
  
Create a new project using postgres and without unit tests.  
```
rails new pm_in_class -db postgresql -T
```
Add the appropriate gems to the gemfile.
```
# Gemfile

gem 'bootstrap-sass'
gem 'haml-rails'
gem 'quiet_assets'
gem 'thin'
gem 'simple_form'
gem 'faker'

group :test do
  gem 'rspec-rails'
  gem 'factory_girl_rails'
end
```
Install the gems, rspec, simple form, and generate a resource for
projects
```
bundle install
rails g rspec:install
rails generate simple_form:install
rails generate resource project title description:text
```
Let's start testing with out projects controller.
```
# spec/controllers/projects_controller_spec.rb

require 'spec_helper'

describe ProjectsController do
  let(:project) { creatre(:project) }
  describe "#index" do

    it "assigns projects instance var that includes poject>" do
      project
      get :index
      expect(assigns(:projects)).to include project
    end
  end

  describe "#new" do
    # before   { get :new }
    # specifiy { assigns(:project).should be }
    # it       { expect(response).to render_template(:new) }

    it "assigns a new project instance var" do
      get :new
      expect(assigns(:project)).to be_instance_of(Project) # ... to
be(true)
    end

  end

  describe "#create" do             # note: describe and context are the
same

    context "with valid params" do
      def valid_request
      post :create, project: {title: "some valid title", description:
"valid description"}
      end
      it "creates a project record in the database" do
        expect(response).to redirect_to(Project.last)
      end
    end

    context "with invalid params" do
      def invalid_request
        post :create, project: {title: "", description:
"valid description"}
      end
      it "does not create a project in the db" do
        expect { invalid_request }.to_not change { Project.count }
      end
      it "renders new template" do
        invalid_request
        expect(response).to render_template(:new)
      end
    end

  end

  describe "#edit" do
    it "assigns an instance variable with project whose id is passed" do
      get :edit, id: project_id
      expect(assigns(:project)).to eq(project)
    end
  end

  describe "#show" do
    
  end

  describe "#update" do
    context "with valid params" do
      def valid_request
        patch :update, id: project.id, project {title: "Some new title"}
      end

      it "changes title to the new passed title" do
        valid_request
        project.reload
        expect(project.title).to eq("Some new title")
      end

      it "redirects to show page" do
        expect(response).to redirect_to(project)
      end
    end

    context "with invalid params" do
      def invalid_request
        patch :update, id: project.id, project: {title: ""}
      end

      it "doesn't change the title" do
        old_title = project.title
        invalid_request
        project.reload
        expect(project.title).to eq(old_title)
      end

      it "renders edit template" do
        expect(resonse).to render_template(:edit)
      end
    end
  end

  describe "#delete" do
    it "removes project from the database" do
      projec
      expect { delete :destroy, id: project.id }.to change { Project.count }.by(-1)
    end

    it "redirect to project listing page" do
      delete :destroy, id: project.id
      expect(response).to redirect_to projects_path
    end
  end

end

```
Add this to the config section of your spec helper, so we can use
Factory Girl methods
```
#spec /spec_helper.rb

  config.include FactoryGirl::Syntax::Methods

```

```
# spec/factories/project.rb
FactoryGirl.define do
  factory :project
  title Faker::Lorem.sentence(5)
  text  Faker::Lorem.sentence(25)
end
```
```
# app/controllers/projects_controller.rb

class ProjectsController
  before_action :find_project, only: [:edit, :show]

  def index
    @project = Project.all
  end

  def new
    @project = Project.new
  end

  def create @project = Project.new(project_params)
    if @project.save
      redirect_to @project, notice: "project created"
    else
      render :new
    end
  end

  def edit
  end

  def show
    @pending_tasks = @project.tasks.pending
    @completed_tasks = @project.tasks.completed
  end

  def update
    if @project.update_attributes(project_params)
      redirect_to @project, notice: "project updated"
    else
      render :edit
    end
  end

  def destroy
    @project.destroy
    redirect_to projects_path
  end

  private

  def project_params
    params.require(:project).permit([:title, :description])
  end

  def find_project
    @project = Project.find(params[:id])
  end
end
```
```
rake db:create db:migrate
```
```
# app/models/project.rb
class Project
  validates :title, presence: true

end
```

Now that we've done the controller, let's do some views!
```haml
/ 
= render projects
```
```haml
%h1 Welcome to PM

= link_to "New Project", new_project_path
```
```haml
= simple_form_for @project, html: {class: "form-horizontal"} do |f|
  / ...
```
```haml
/ show
%h1= @project.title
%p= @project.description

= link_to "Edit", edit_project_path(@project)
|
= link_to "delete, @peoject, method: :delete

= simple_form_for [@project, Task.new] do |f|
  = f.input :title
  = f.submit "save task", class: "btn btn-primary"

.row
  .col-sm-12
    .col-sm-6
      Pending Tasks
      .well#pending-tasks
    .col-sm-6
      Completed Tasks
      .well#completed-tasks
```
```ruby
require 'spec_helper'

describe TasksController do
  let(:project) { create(:project) }
  describe "#create" do

    context "with valid request" do
      def valid_request
        post :create, project_id: task: {title: "some valid title"}
      end
      it "creates a task for the project" do
        expect { valid_request }.to change { project.tasks.count }.by(1)
      end

      it "assigns the task to the project" do
        valid_request
        expect(Task.last.project).to eq(project)
      end

      it "render success json message" do
        valid_request
        expect(response.body).to include "success"
      end

      it "sends back the title" do
        valid_request
        parsed_body = JSON.parse(response.body)
        expect(parsed_body["url"]).to eq(project_task_path)
      end
    end

    def "with invalid request" do
      def invalid_request
        post :create, project_id: project.id, task: {title: ""}
      end

    it "doesn't create a task" do
      expect { invalid_request }.to_not change(task.count)
    end

    it "renders failire status" do
      invalid_request
      parsed_body = JSON.parse(response.body)
      expect(parsed_body)["status"].to eq("failure")
    end

    it "sends error message" do
      invalid_request
    end
  end

  describe "#update"
    let(:task) { create(:task, project: project, completed: false) }
    it "should update the changed value in the database" do
      patch :update, project_id: task.project.id, task.id, task:
{completed: true}
      task.reload
      expect(task.compeleted).to eq(true)
    end
  end
end

```
```ruby
class TasksController < ApplicationControlelr

  before_action :find_project

  def create
    @task = @project.tasks.new(task_params)
    if @task.save
      render json: {status: "success"}
    else
      render :new, alert: "Not created, yo!"
    end
  end

  def update
    @task = Task.find(params[:id])
    if @task.update_attributes(task_params)
      
    else
    end
  end

  private

  def task_params
    params.require(:task).permit(:title)
  end

  def find_project
    @project = Project.find(params[:project_id])
  end
end
```
```ruby
class Task < ActionController
  validates :title, presence: true

end
```
```coffee
# javascripts/something.js.coffee

$ ->
  $(document).on "submit", "#new_task, ->

    $.ajax
      method: "post"
      url: $(@).attr("action")
      data:
        task:
          title: $("#task_title").val()
      errors: ->
        alert("Task didn't create. Please reload page.")
      success: (data) ->
        if data.status == "success"
          remplate = $("#task-template").html()
          rendered = Mustache.render(template, {title: data.title})
          $("#pending-tasks").append(rendered)
        else
          alert("failure")
    false

  $(".draggable").draggable
    appendTo: "alert-info"

  $(".droppable").droppable
    hoverClass: "alert-info"
    drop: (event, ui) ->
      $(ui.draggable).css("left"), 0);
      $(ui.,draggable).css("top", 0);
      $(@).prepend($(ui.draggable));
      if $(@).attr("id") == "completed-tasks"
        completed = true
      else
        completed = false
      update_task_status $(ui.draggable).data("url"),
                         $(@).attr("id") == "completed-tasks"

update_task_status = (url, status) ->
  $.ajax
    method: "patch"
    url: url
    data:
      task:
        completed: status
    error: ->
      alert("Something went wrong please refresh page")")
    success: (data) ->
      unless data.status == "success"
        alert("soemthing went wrong please refresh!")
```
Add jquery ui to your gemfile
```ruby
# Gemfile

gem 'jquery-ui-rails'

```
bundle install and then restart your server, because you added a new
gem.  
  
Now, we can use [draggable](http://jqueryui.com/draggable/) and [droppable](http://jqueryui.com/droppable/).  
  
```
# spec/models/task_spec.rb

require "spec_helper"

describe Task do
  let(:completed_task) { create(:task, completed: true) }
  let(:pending_task) { create(:task, compeleted: false) }

  describe "#completed" do
    subject { Task.completed }

    it { should include(completed_task) }
    it { should_not include(pending_task) }
  end

  describe "#pending" do
    subject { Task.pending }

    it { should include(pending_task) }
    it { should_not include(completed_task) }
  end

end
```
```
FactoryGirl.define do

  factory :task do
    association :project, factory: :project
    title Faker::Company.bs
    completed false
  end

end
```
```ruby
class Task < ActiveRecord::Base
  belongs_to :project

  validates :title, presence :true



end