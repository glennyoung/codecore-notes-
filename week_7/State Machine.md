# State Machine
[state machine](https://github.com/pluginaweek/state_machine) | [Issue 295](https://github.com/pluginaweek/state_machine/issues/295) | [canceled or cancelled](http://www.quickanddirtytips.com/education/grammar/canceled-or-cancelled)    
Add the state machine gem to your Gemfile
```ruby
gem 'state_machine'
```  
To resolve issue 295, we can create a file in our initializers directory located within our config directory.  
```ruby
# config/initializers/state_machine_patch.rb

# Rails 4.1.0.rc1 and StateMachine don't play nice
# https://github.com/pluginaweek/state_machine/issues/295

require 'state_machine/version'

unless StateMachine::VERSION == '1.2.0'
  # If you see this message, please test removing this file
  # If it's still required, please bump up the version above
  Rails.logger.warn "Please remove me, StateMachine version has changed"
end

module StateMachine::Integrations::ActiveModel
  public :around_validation
end
```
generate a migrattion
```
rails g migration add_state_to_campaigns state
```
Open the migration file and add an index
```
class AddStateToCampaigns < ActiveRecord::Migration
  def change
    add_column :campaigns, :state, :string

    add_index :campaigns, :state
  end
end
```
then `rake db:migrate`  
```ruby
# app/models/campaign.rb

class Campaign < ActiveRecord::Base
  belongs_to :user

  scope :published, -> { where(state: :published) }

  state_machine :state, initial: :draft do

    event :publish do
      transition draft: :published
    end

    event :complete do
      transition published: :target_met
    end

    event :expire do
      transition target_met: :succeeded, published: :failed
    end

    event :cancel do
      transition [:draft, :published, :target_met] => :canceled
    end

    after_transition on: :canceled, do: :refund_all_pledges
  end

  def refund_all_pledges
    # code to schedule bg task to refund all
  end
end
```
In Rails console
```
Campaign.update_all(state: :draft)
c = Campaign.first
c.draft?
c.state == "draft"
c.published?
c.publish                       # auto publish
c.complete
c.expire
c.cancel
```
Modify the welcome controller to only display published campgains
```ruby
# app/controllers/welcome_controller.rb

class WelcomeController < ApplicationController
  def index
    @campaigns = Campaign.published.paginate(page: params[:page], per_page: 20).order("created_at DESC")
    respond_to do |format|
      format.html { render :index }
      format.js   { render :index }
    end
  end
end
```
Add an index method to the campaigs controller
```ruby
# app/controllers/campaigns_controller.rb

class CampaignsController < ApplicationController
  before_action :authenticate_user!, except: :show

  def index
    @campaigns = current_user.campaigns
  end
  
  # ...

end
```
Add a campaign index view
```haml
/ app/views/campaigns/index.html.haml

- @campaigns.each do |c|
  .well
    %h4= c.title
    %p= c.details.truncate(50)
    - if c.draft?
      = button_to "Publish", publish_campaign_path(campaign), class: "btn btn-default"
    %p
      =link_to "View", c, class: "btn btn-default", data: {toggle: "modal", target: "#campaign" }, remote: true
```
Add a link to the user info partial if the user is signed in
```haml
  = link_to "My campaigns", campaigns_path, class: 'btn btn-default'
```
Add to the routes
```ruby
# config/routes.rb

# ...

  resources :campaigns do
    patch :publish, on: :member
  end
  
# ...
  
```

Add a publish method to the campaigns controller
```ruby
# app/controllers/campaigns_controller.rb

# ...

  def publish
    @campaign = Campaign.find(params[:id])
    if @campaign.publish
      redirect_to campaigns_path, notice: "Campaign published successfully."
    else
      redirect_to campaigns_path, alter: "Campaign publishing failed: #{@campaign.errors.full_messages}"
    end
  end

# ...

```
Now, to give campaigns useful labels for their states, let's add a method for label class to our campaign helper
```ruby
# app/helpers/campaigns_helper.rb

module CampaignsHelper

  def label_class(state)
    case state.to_sym
    when :draft then "label label-default"
    when :published then "label label-primary"
    when :target_met then "label label-info"
    when :succeeded then "label label-success"
    when :failed then "label label-danger"
    when :cancelled then "label label-danger"
    end
  end
end
```
And modify the campaign index view to make use of the labels.
```haml
/ app/views/campaigns/index.html/haml

- @campaigns.each do |c|
  .well
    %h4= c.title
    %p= c.details.truncate(50)
    %p{class: label_class(campaign.state)}
      = c.state.capitalize
    - if c.draft?
      = button_to "Publish", publish_campaign_path(campaign), class: "btn btn-default", method: :patch
    %p
      =link_to "View", c, class: "btn btn-default", data: {toggle: "modal", target: "#campaign" }, remote: true

```