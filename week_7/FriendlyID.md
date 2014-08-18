# FriendlyID
To stop campaigns from using AJAX, just comment out the coffeescript
```coffeescript
# app/assets/javascripts/campaigns.js.coffee

# Place all the behaviors and hooks related to the matching controller here.
# All this logic will automatically be available in application.js.
# You can use CoffeeScript in this file: http://coffeescript.org/

#$ ->
#  $(document).on "click", ".pagination a", ->
#    $(".pagination a").css("opacity", 0.25)
#    $.getScript($(@).attr("href"))
#    false

```

Let's open up the comapaign model and define a method that parameterizes the ID.
```ruby
# app/models/campaign.rb

# ...

  def to_param
    "#{id}-#{title}".parameterize
  end
  
# ...

```
This will give us a little nicer of a URL in the browser. However, there are still some downsides to using this approach, and a more popular option is to use the [friendly ID](https://github.com/norman/friendly_id) gem.
```ruby
# Gemfile

gem 'friendly_id', '~> 5.0.0'
```
Add a migration to add slug to campaigns
```rake
rails generate friendly_id
rails generate migration add_slug_to_campaigns slug
```
Open up the migration file and add an index (unique: true is optional).  
```ruby
class AddSlugToCampaign < ActiveRecord::Migration
  def change
    add_column :campaigns, :slug, :string, unique: true
    add_index :campaigns, :slug
  end
end
```
Migrate the file `rake db:migrate`  
  
Now, we can add to the campaign model 
```ruby
# app/models/campaign.rb

# ...

  extend FriendlyId
  friendly_id :title, use: :slugged
  
# ...
```
After you install a gem, you should restart your server.

in rails c try.
```
# If you're adding FriendlyId to an existing app and need
# to generate slugs for existing users, do this from the
# console, runner, or add a Rake task:

Campaign.find_each(&:save)
```
Now, in the campaigns controller change find to find.friendly for the show and publish methods
```ruby
# app/controllers/campaigns_controller.rb

# ...

  def show
    @campaign = Campaign.friendly.find(params[:id])
    @commentable = @campaign
    @comment = Comment.new
    respond_to do |format|
      format.html { render :show }
      format.js { render :show }
    end
  end
  
# ...

  def publish
    @campaign = Campaign.friendly.find(params[:id])
    if @campaign.publish
      redirect_to campaigns_path, notice: "Campaign published successfully."
    else
      redirect_to campaigns_path, alter: "Campaign publishing failed: #{@campaign.errors.full_messages}"
    end
  end
  
# ...

```
## Handling Nested Forms
```
rails generate model reward_level title details:text amount:integer campaign:references
```
Add to campaign model a has many relationship, and accepts nested attributes for reward levels.
```ruby
# app/models/campaign.rb

# ...

  has_many :reward_levels, dependent: :destroy
  accepts_nested_attributes_for :reward_levels
  
# ...
  
```

Now that our campaign form can accept nested attributes for reward levels, let's add the fields to the new form for campaigns
```haml
/ app/views/campaigns/new.html.haml

= simple_form_for @campaign, html: { class: "form-horizontal" } do |f|
  = f.input :title
  = f.input :details
  = f.input :target
  = f.input :end_date

  = f.fields_for :reward_levels do |rl|
    = rl.input :title
    = rl.input :details
    = rl.input :amount
    %hr

  = f.submit "Submit", class: "btn btn-default"
```
In the new action of the campaigns controller, you can define how many times to build the nested form, for example 3 times. Also, we need to add the reward levels attributes parameters to the accepted parameters for our campaign params.
```ruby
# app/controllers/campaigns_controller.rb

# ...

  def new
    @campaign = Campaign.new
    3.times { @campaign.reward_levels.build }
  end
  
# ...

  def campaign_params
    params.require(:campaign).permit(:title, :details, :full_street_address, :target, :end_date, {reward_levels_attributes: [:title, :details, :amount, :_destroy, :id]})
  end

# ...
  
```
Now that it's passing all these fields, it's going to try to save them all in the database. Even if there is nothing put into the reward levels form fields.
  
In the campaign model we can add to the accepts nested attributes a reject if clause that checks for amount, title, and details to be present in the reward level params. And a private method to make sure there is at least one reward level given.  
```ruby
# app/models/campaign.rb

# ...

  accepts_nested_attribues_for :reward_levels,
    reject_if: proc {|x| x[:amount].blank? && x[:title].blank? && x[:details].blank? }
    
# ...

  private

  def has_a_reward_level
    if reward_levels.size < 1
      errors.add(:title, "Must put at least one reward level")
    end
  end
```
We can add some validations for these fields to our reward level model
```ruby
# app/models/reward_level.rb

class RewardLevel < ActiveRecord::Base
  belongs_to :campaign
  validates_presence_of :title, :details, :amount
end

```
Add a lamda to build the empty form fields for reward levels if create fails.  
```ruby
# app/controllers/campaigns_controller.rb

# ...

  def create
    @campaign = Campaign.new(campaign_params)
    if @campaign.save
      redirect_to root_path, notice: "success"
    else
      (3 - @campaign.reward_levels.length).times { @campaign.reward_levels.build }
      render :new
    end
  end
  
# ...

```
Add a way to display meaningful errors to the campaigns new view.   
```haml
/ app/views/campaigns/new.html.haml

- if @campaign.errors.has_key?(:base)
  .alert.alert-danger= @campaign.errors[:base].join(", ")
```

Let's use another interesting thing that accepts nested attributes allows destroy to be true.
```ruby
# ...

 accepts_nested_attributes_for :reward_levels, allow_destroy: true,
    reject_if: proc {|x| x[:amount].blank? && x[:title].blank? && x[:details].blank? }
    
#...
```

Add a field to the form
```haml
/ app/views/campaigns/new.html.haml

- if @campaign.errors.has_key?(:base)
  .alert.alert-danger= @campaign.errors[:base].join(", ")

= simple_form_for @campaign, html: { class: "form-horizontal" } do |f|
  = f.input :title
  = f.input :details
  = f.input :target
  = f.input :end_date

  = f.fields_for :reward_levels do |rl|
    = render "reward_level_fields", f: rl

  = f.submit "Submit", class: "btn btn-default"
```
And a form partial for reward level fields in the campaigns view directory
```haml
/ app/views/campaigns/_reward_level_fields.html.haml

%fieldset
  = f.input :title
  = f.input :details
  = f.input :amount
  = f.input :_destroy, as: :hidden
  = link_to "Remove", "javascript:void(0)", class: "remove-reward-level"
  %hr
  = link_to "Add Reward Level", "javascript: void(0)", class: "btn btn-default"
```
Now, if we want to be able to remove these extra fields from the view using javascript, let's pull up the campaigns javascript file.  
```coffeescript
# app/assets/javascripts/campaigns.js.coffee

  $(document).on "click", ".remove-reward-level", ->
    $(@).parents("fieldset").fadeOut(1000)
    $(@).parents("fieldset").find("input[type=hidden]").val("1")
```

## Soemthing cool from Rails Casts
Let's add a method to our application helper
```ruby
# app/helpers/application_helper.rb

# ...

  # name        => label of button
  # f           => form object
  # assocaition => nested_attributes e.g. reward_evels
  def link_to_add_fields(name, f, association)
    new_object  = f.object.send(association).new
    id          = new_object.object_id

    # f.fields_for :reward_levels
    fields = f.fields_for(association, new_object, child_index: id) do |rl|
      render(association.to_s.singularize + "_fields", f: rl)
    end
    link_to(name, "javascript:void(0)", class: "add_fields", data: {id: id, field: fields.gsub("\n", "")})
  end
  
# ...

```
So, let's change the link in the campaign new view to make use of this method we have just defined in our application helper.  
```haml
/ app/views/campaigns/_reward_level_fields.html.haml

%fieldset
  = f.input :title
  = f.input :details
  = f.input :amount
  = f.input :_destroy, as: :hidden
  = link_to "Remove", "javascript:void(0)", class: "remove-reward-level"
  %hr
  /= link_to "Add Reward Level", "javascript: void(0)", class: "btn btn-default"
  = link_to_add_fields("Add Reward Level, f, :reward_levels")

```
And add this to the javascript file
```coffeescript

  $(document).on "click", ".add_fields", ->
    time = new Date().getTime()
    regex = new RegExp($(@).data("id"), "g")
    $(@).before($(@).data("fields").replace(regex, time))
    false

```

***Aside*** (rails c)
```
my_array = [1, 2, 3, 4]
method_name = :length
my_array.send(method_name)

@campaign = Campaign.first
association = :reward_levels
@campaign.send(association)
```