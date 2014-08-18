# Stripe  
[Stripe API](https://stripe.com/docs/api/ruby) | [Stripe Rails](https://github.com/thefrontside/stripe-rails) | [Stripe Guide (rails)](https://stripe.com/docs/checkout/guides/rails) | [Stripe Ruby](https://github.com/stripe/stripe-ruby) | [Example Card Numbers](https://stripe.com/docs/testing#cards)  
  
[Sign up](https://manage.stripe.com/register) for a Stripe account. You only need to enter your banking information if you want to take it live.  
  
Go to your account > your account settings > api keys
  

Add the stripe gem to your Gemfile
```ruby
# Gemfile

gem 'stripe', :git => 'https://github.com/stripe/stripe-ruby'

```
Create a file under config > initializers
```ruby
# config/initiliazers/keys_for_stripe.rb

```
```ruby
# config/initializers/setup_stripe.rb
Stripe.api_key = ENV["STRIPE_SECRET_KEY"]

```
And add to your git ignore file
```ruby
.gitignore



```
Add to your applications layout, and add to the javascripts (check the Javascript API docs for details, something like this: https://js.stripe.com/v1)
```haml
/
/ ...

= tag :meta, name: "stripe-key", content: ENV["STRIPE_PUBLISHABLE_KEY"]

/ ...

/ or in the guide like this

%script.stripe-button{"data-key" => Rails.configuration.stripe[:publishable_key], src: "https://checkout.stripe.com/checkout.js"}
```
Generate a rails resource for orders  
```
rails g resource order user:references reward_level:references stripe_txn_id
```
In the routes, because we want to create an order for a reward level, we can nest the resources for orders under reward levels
```ruby

# routes.rb

# ...

resources :reward_levels, only" [] do
  resources :orders
end

# ...

```
```ruby
class OrdersController < ApplicationController

  before_action :authenticate_user!
  before_action :find_rewqard_level

  def new
    @order = Order.new
  end

  def create
    # setup a test charge
    Stripe::Charge.create(
       :amount => 400,
       :currency => "cad",
       :card => params[:order][:stripe_card_token],
       :description => "Test Charge"
     )
     render text: "success" 
  end

  private

  def find_reward_level
    @reward_level = RewardLevel.find(params[:reward_level_id])
  end

end
```
Let's create a delegate for the reqard level model. This way, we can make one call, instead of saying `reward_level.campaign.title`, we can say `reward_level.campaign_title`. If we don't put prefix, it's just going to be `reward_level.title` However, the problem with this is that reward level already has a title.  
```ruby
class RewardLevel < ActiveRecord::Base
  belongs_to :campaign

  validates_presence_of :title, :details, :amount

  delegate :title, to: :campaign, prefix: true
  
end
```
If you add an attribute accessor to your order you can access the card number from your form.

```ruby
class Order < ActiveRecord::Base
  belongs_to :user
  belongs_to :reward_level

  attr_accessor :card_number, :cvc, :card_month, :card_year, :stripe_card_token

end
```
Create a new orders view
```haml
/ new.html/haml

:

%h1 Pledge #{@reward_level.amount} for #{@reward_level.campaign_title} 

= simple_form_for [@reward_level, @order] do |f|
  = f.input :stripe_card_token, as: :hidden
  = f.input :card_number
  = f.input :cvc
  = f.input :card_month, as: :date, order: [:month]
  = f.input :card_year, as: :date, order: [:year], start_year: Date.today.year, end_year: Date.today.year + 5
  = f.submit "Pledge!", class: "btn btn-default, data: { disable_with: "Submitting..." }
``` 
Let's add a link beside reward levels that says "Pledge" that will take the user to the new order view
```haml
/ app/views/campaigns/show.html.haml

#stripe-error

/ ... campaign.reward_levels.each do |reward_level|

= link_to "Pledge", new_reward_level_path(reward_level), class: "btn btn-default"

/ ...

```
This way, users cannot double submit information.  
```coffeescript
# app/assets/javascripts/orders.js.coffee

$ ->
  return if typeof Stripe == "undefined"
  Stripe.setPublishableKey($("meta[name='stripe-key']").attr("content")

  $(document).on "submit", "#new_order", ->
    $("input[type=submit]").attr("disabled", true)
    processCard()
    false

processCard = -> 
  card = 
    number: $("#order_card_number").val()
    cvc: $("#order_cvc").val()
    expMonth: $("#order_card_month_2i").val()
    expYear: $("#order_card_year_1i").val()
  
  Stripe.createToken(card, handleStripeResponse)

handleStripeResponse = (status, response)->
  if status == 200
    $("#order_card_number").val("")
    $("#order_cvc").val("")
    $("#order_card_month_2i").val("")
    $("#order_card_year_1i").val("")
    $("#order_stripe_card_token").val(response.id)
    $("#new_order")[0].submit()
    response.id
  else
    $("#stripe-error").text(response.error.message)
    $("input[type=submit]").attr("disabled", false)
```
In config > initializers > filter parameter logging add some fields for the credit card info
```ruby
# config/initializers/filter_parameter_logging.rb

Rails.application.config.filter_parameters += [:password, :card_number, :cvc, :card_month, :card_year]
```
Add a migration to add fields to the user for stripe columns to users
```
rails generate migration add_stripe_columns_to_users stripe_customer_id stripe_card_last4 stripe_card_type
```
Open up the migrations file and look at it. Now instead of creating a charge right away when we create a new order, we want to create a new customer.  
```ruby
# app/controllers/orders_controller.rb

class OrdersController < ApplicationController

  before_action :authenticate_user!
  before_action :find_rewqard_level

  def new
    @order = Order.new
  end

  def create
    token = params[:order] ? params[:order][:stripe_card_token] : ""
    service = Order::CreateOrder.new(user: current_user,
                      stripe_token: token,
                      reward_level: @reward_level)
    if service.call
      redirect_to @reward_level.campaign, notice: "Thanks for pldeging"
    else
      @order = service_order
      render :new
    end
  end

  private

  def find_reward_level
    @reward_level = RewardLevel.find(params[:reward_level_id])
  end

end
```
Create a new service
```
# services/order/create_order.rb

class Order::CreateOrder

  include Virtus.model

  attribute :stripe_token, String
  attribute :user, User
  attribute :reward_level, RewardLevel
  attribute :order, Order

  def call
    build_order
    begin
      create_customer unless user.stripe_customer_id
      @order.stripe_txn_id = charge_customer.id
    rescue Stripe::CardError => e
      @order.errors.add(:base, "Card Problem")
      return false
    end
    @order.save
  end

  private

  def build_order
    @order = Order.new
    @order.user = user
    @order.reward_level = reward_level
  end

  def create_customer
    service = Stripe::CreateCustomer.new(user: user,
                        stripe_token: stripe_token)
    service.call
  end

  def charge_customer
    Stripe::Charge.create(
            :amount => reward_level.amount * 100,
            :currency => "cad",
            :customer => user.stripe_customer_id,
            :description => "pledge for #{reward_level}"
          )
  end

end

```
and another service object called stripe
```ruby
# models/services/stripe/create_customer.rb


class Stripe::CreateCustomer

  include Virtus.model

  attribute :user, User
  attribute :stripe_token, String

  def call
    customer = Stripe::Customer.create(
               description: default_description,
               card:        stripe_token
    )
    user.stripe_customer_id = customer.id
    user.stripe_card_last4 = customer.card.data[0].last4
    user.stripe_card_tyupe = customer.cards.data[0].type
    user.save
  end

  private

  def default_description
    "customer for #{user.email} | id: #{user.id}"
  end
end
```
And now we can modify the create action in the orders controller
```ruby
# app/controllers/orders_controller.rb

# ...

  def create
    service = Order::CreateOrder.new(user: current_user,
            stripe_token: params[:order][:stripe_token],
            reward_level: @reward_level)
    if service.call
      redirect_to @reward_level.campaign, notice: "Thanks for pldeging"
    else
      @order = .service_order
      render :new
    end
  end

# ...
```

Add to your orders new view
```haml
/ app/views/orders/new.html.haml

- if @order.errors[:base].present?
  .alert.alert-danger= @order.errors[:base].join(", ")

- if current_user.stripe_customer_id
  = simple_form_for [@reward_level, @order, html: {id: "exisiting_card"} ] do |f|
    = f.input :stripe_card_token, as: :hidden, input_html: {value: ""}
    .well
      = current_user.stripe_card_type
      **** **** **** #{current_user.stripe_card_last4
      = f.submit "Charge This Card", class: "btn btn-default"

- else
  = simple_form_for ...

```
And then you can add an 'if' order card number has a length process card to the orders coffeescript
```coffeescript

$ ->
  return if typeof Stripe == "undefined"
  Stripe.setPublishableKey($("meta[name='stripe-key']").attr("content")

  $(document).on "submit", "#new_order", ->
    $("input[type=submit]").attr("disabled", true)
    if $("#order_card_number").length
      processCard()
      false

processCard = ->
  card =
    number: $("#order_card_number").val()
    cvc: $("#order_cvc").val()
    expMonth: $("#order_card_month_2i").val()
    expYear: $("#order_card_year_1i").val()

  Stripe.createToken(card, handleStripeResponse)

handleStripeResponse = (status, response)->
  if status == 200
    $("#order_card_number").val("")
    $("#order_cvc").val("")
    $("#order_card_month_2i").val("")
    $("#order_card_year_1i").val("")
    $("#order_stripe_card_token").val(response.id)
    $("#new_order")[0].submit()
    response.id
  else
    $("#stripe-error").text(response.error.message)
    $("input[type=submit]").attr("disabled", false)
```