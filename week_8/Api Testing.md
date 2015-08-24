[jBuilder gem](https://github.com/rails/jbuilder) | [mustache.js](https://github.com/janl/mustache.js/)

```ruby
# Gemfile

gem 'factory_girl_rails'
gem 'rspec-rails'
bundle install
rails generate rspec install
```
```ruby
# spec/spec_helper.rb
# A new comment 

RSpec.configure do |config|
  config.include FactoryGirl::Syntax::Methods
```
Inside spec create a controllers folder and an api folder, then a v1 folder
```ruby
# spec/controllers/api/v1/campaigns_controller.rb

require 'spec_helper'

describe Api::V1::CampaignsController do
  render_views

  describe "fetching all campaigns" do
    let!(:campaign_1) { create(:campaign) }
    let!(:campaign_2) { create(:campaign) }

    before do
      get :index, format: :json
    end

    specify { response.body.should include(campaign_1.title) }
    specify { response.body.should include(campaign_2.title) }
    specify { response.body.should include(campaign_1.details) }
    specify { response.body.should include(campaign_2.details) }

  end

  describe "fetching sinle campaign" do
    let(:campaign) { create(:campaign) }

    before do
      get :show, id: campaign.id, format: :json
    end

    it "incluides campaign's title" do
      body_json = JSON.parse(response.body)
      expect(body_json["title"]).to eq(campaign.title)
    end
  end
end
```
```ruby
# spec/controllers/campaigns_controller_spec.rb

class Api::V1::CampaignsController < ApplicationController

  def index
    @campaigns = Campaign.paginate(page: params[:page], per_page: 12)

  end
end
```
```ruby
# spec/factories/campaign.rb

FactoryGirl.define do

  factory :campaign do
    association :user, factor: :user
    title Faker::Company.bs
    detauils Faker::Lorem.paragraph
    target 100000
    end_date (Time.now + 20.days)
  end

end
```
```ruby
# spec/factories/user.rb

FactoryGirl.define do

  factory :user do
    sequence(:email) { |x| "some_email#{x}@gmail.com" }
    password "somevalidpassword123"
  end

end
```
Add a path in your routes to have campaigns
```ruby
# config/routes.rb

Rails.application.routes.draw do

  root 'welcome#index'

  get "signup" => "user#new", as: :signup

  resources :users

  namespace :api, defaults: { format: :json } do
    namespace :v1 do
      resources :campaigns
    end
    # scope module: :v1 do
    #  resources :campaigns
    # end
  end

end

```
```jbuilder
# app/views/api/v1/index.json.jbuilder

json.array! @campaigns do |campaign|
  json.id campaign.id
  json.title campaign.title
  json.details campaign.details
end

# [ { title: " ..."}, {title: "..."}]
```
```jbuilder
# app/views/api/v1/show.json.jbuilder

json.title @campaign.title
json.details @campaign.details
json.target @campaign.target
json.end_date @campaign.end_date.strftime("")
```
Rails c
```bash
@campaign = Campaign.limit(10)
@campaign.to_json
```
```ruby
# app/controllers/application_controller.rb

class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception

  after_action :set_access_controller_headers

  def set_access_controller_headers
    headers['Access-Control-Allow-Origin'] = '*'
    headers['Access-Control-Request-Method'] = '*'
  end
end
```
Let's make a new application!
```
rails new crowdfunding_client -T
```
Include the [mustache js library](https://raw.githubusercontent.com/janl/mustache.js/master/mustache.js).
```erb
<!-- app/views/welcome/index.html.erb -->

<script id="campaign-listing-template" type="x-tmpl-mustache">

  <div class="campaign">
    <h2 class="title">
      <a href="javascript:void(0);" class="title" data-id="{{id}}">{{title}}</a>
    </h2>
    <p>{{details}}</p>
    <h3>{{target}}</h3>
    <hr>
  </div>
</script>

<script id="single-campaign-display" type="x-tmpl-mustache">
  <h1>{{title}}</h1>
  <p>Details: {{details}}</p>
  <p>Target: $ {{target}}</p>
  <p>End Date: {{end_date}}</p>
  <a href="javascript:voice(0);" class="back">Back</a>
</script>

<h1>Welcome to Fund.sy</h1>

<div id="single-campaign-display"></div>

<div id="campaigns"></div>

<div id="single-campaign-container"></div>
```
```coffeescript
# app/assets/javascripts/welcome.js.coffee

$ ->
  s.ajax
    url: "http://localhost:3000/v1/campaigns"
    dataType: "json"
    method: "get"
    error: ->
      alert("ERROR")
    success: (data) ->
      template = $("#campaign-listing-template").html
      Mustache.parse(template)
      for campaign in data
        rendered_template = Mustache.render()
        $("#campaigns").append rendered_template

  $("#campaigns").on "click", ".title", ->
    $.ajax
      url: "http:.//localhost:3000/v1/campaigns/" + $(@).data("id")
      dataType: "json"
      method: "get"
      error: ->
        alert("ERROR")
      success: (data) ->
        template = $("#single-campaign-display").html()
        rendered = Muastache.render(template, data)
        $("#single-campaign-container").hide()
        $("#single-campaign-container").html rendered
        $("#campaigns").fadeOut 700, ->
          $("#single-campaign-container").fadeIn(300)
    false

  $("#single-campaign-container").on "click", ".back", ->
    $("#single-campaign-container").fadeOut 700, ->
       $("#campaings").fadeIn 300
```
In the crowdfunding app generate a new API model
```ruby
rails generate model api_key access_token user:references
```
Everytime you genereate an api key, I need to generate a unique access token.
```ruby
# app/models/api_key.rb

class ApiKey < ActiveRecord::Base
  belongs_to :user

  before_create :generate_unique_access_token

  private

  def generate_unique_access_token
    begin
      self.access_token = SecureRandom.hex
    end while self.class.exists?(access_token: access_token)
  end
end
```
```
User.exists?(email: 'tam@codecore.ca')
```
```ruby
# app/models/user.rb

class User < ActiveRecord::Base
  has_secure_password
  has_one :api_key

  validates :email, presence: true, uniqueness: true, email_format: true
  before_create :generate_api_key

  private

  def generate_api_key
    self.api_key = ApiKey.new
  end

end
```
Create a new user and use the access token that is generated on create to fill out your application coffee script file.  
```coffeescript
# crowdfunder: app/assets/javascripts/application.js.coffee

$ ->
  s.ajax
    url: "http://localhost:3000/v1/campaigns"
    dataType: "json"
    method: "get"
    access_token: "665ad0a5b85957ac576677c62ffc184c"
    error: ->
      alert("ERROR")
    success: (data) ->
      template = $("#campaign-listing-template").html
      Mustache.parse(template)
      for campaign in data
        rendered_template = Mustache.render()
        $("#campaigns").append rendered_template

  $("#campaigns").on "click", ".title", ->
    $.ajax
      url: "http:.//localhost:3000/v1/campaigns/" + $(@).data("id")
      dataType: "json"
      method: "get"
      data:
        access_token: "665ad0a5b85957ac576677c62ffc184c"
      error: ->
        alert("ERROR")
      success: (data) ->
        template = $("#single-campaign-display").html()
        rendered = Muastache.render(template, data)
        $("#single-campaign-container").hide()
        $("#single-campaign-container").html rendered
        $("#campaigns").fadeOut 700, ->
          $("#single-campaign-container").fadeIn(300)
    false

  $("#single-campaign-container").on "click", ".back", ->
    $("#single-campaign-container").fadeOut 700, ->
       $("#campaings").fadeIn 300

```
