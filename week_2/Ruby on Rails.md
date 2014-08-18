# Ruby on Rails  
  
Rails is simple, but the knowledge builds up quickly. We're going to be going over some projects. In class, we'll be covering a project called "Awesome Answers". Please try to follow along with this, as we will be continuing from where we left off, on a daily basis.  
  
There will also be a project for a "Project Management Tool" that will be applying the principles learned through exercises with "Awesome Answers."  
  
**Rails**
  * saves you a lot of time
  * has a large community
  * has many gems
  * is a well known framework  
  
Rails was created by David Heinemeir Hansson ([DHH](https://twitter.com/dhh)) of [37 Signals](https://37signals.com/). Github uses rails, groupon, shopify, yellow pages, Basecamp, twitter (at first).  
  
Rails is very opinionated. The core team made decisions to do things a certain way, and so there are many conventions in rails. Rails is a gem, just like Sinatra.  
  
Let's start by installing rails, if you haven't already.  
```bash
gem install rails
# or
gem install rails --no-rdoc
```  
If you specify a gem version with `~>`, it will install the latest stable version, i.e. in the case of rails 4.0.4, it would install up to 4.0.9, but not 4.1.  
```bash
gem 'rails', '~> 4.0.4'
```  
Rails comes with many parts, including actionmailer, so we no longer need the Pony gem, for example.  
  
Create a new rails project called "awesome_answers", using a postgresql database.  
```bash
rails new awesome_answers -d postgresql
```  
cd into the directory rails created called "awesome_answers" and open it up in your favorite text editor.  
```bash
cd awesome_answers
subl .
```  
  
## Files and Structure
```bash
config.ru           # is where we tell the server to require our app.
.gitignore          # here we can list files we do not want git to track, e.g. config/database.yml
config/database.yml # if you are using postgres, or any database other than sqlite, you will need to specify your username
```  
  
**app**: This organizes your application components. It's got subdirectories that hold the view (views and helpers), controller (controllers), and the backend business logic (models).

**app/controllers**: The controllers subdirectory is where Rails looks to find controller classes. A controller handles a web request from the user.

**app/helpers**: The helpers subdirectory holds any helper classes used to assist the model, view, and controller classes. This helps to keep the model, view, and controller code small, focused, and uncluttered.

**app/models**: The models subdirectory holds the classes that model and wrap the data stored in our application's database. In most frameworks, this part of the application can grow pretty messy, tedious, verbose, and error-prone. Rails makes it dead simple!

**app/view**: The views subdirectory holds the display templates to fill in with data from our application, convert to HTML, and return to the user's browser.

**app/view/layouts**: Holds the template files for layouts to be used with views. This models the common header/footer method of wrapping views. In your views, define a layout using the `layout :default` and create a file named default.rhtml. Inside default.rhtml, call `<% yield %>` to render the view using this layout.  
ref: [tutorialspoint](http://www.tutorialspoint.com/ruby-on-rails/rails-directory-structure.htm)  
  
## database.yml
```ruby
# config/database.yml
development:
  adapter: postgresql
  encoding: unicode
  database: awesome_answers_development
  pool: 5
  username: my_mac_username             # in terminal type whoami if you wish to see your Mac username.
  password:

test:
  adapter: postgresql
  encoding: unicode
  database: awesome_answers_test
  pool: 5
  username: my_mac_username           
  password:

# production:                           # because we will deploy to Heroku, which sets its own database, we don't need this
#   adapter: postgresql
#   encoding: unicode
#   database: awesome_answers_production
#   pool: 5
#   username: my_mac_username
#   password:
```  
Make sure you are in your app's directory "awesome_answers" and run the command `rails s` or `rails server` in the terminal. This should start up your rails server, and give a message like this  
```bash
=> Booting WEBrick
=> Rails 4.0.2 application starting in development on http://0.0.0.0:3000
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server
[2014-03-24 10:09:23] INFO  WEBrick 1.3.1
[2014-03-24 10:09:23] INFO  ruby 2.1.0 (2013-12-25) [x86_64-darwin12.0]
[2014-03-24 10:09:23] INFO  WEBrick::HTTPServer#start: pid=90355 port=3000
```  
If you get an error stating the databse does not exist, run `rake db:create`.  
  
## MVC
MVC resources: [video](https://www.youtube.com/watch?v=3mQjtk2YDkM) | [Coding Horror](http://blog.codinghorror.com/understanding-model-view-controller/) | [Better Explained](http://betterexplained.com/articles/intermediate-rails-understanding-models-views-and-controllers/)  
In Sinatra, we had our routes in our controllers, when we did something like  
```ruby
get "/" do
  @tasks = Task.all         # Task is a model
  erb :index                # index.erb is a view
end
```  
In Rails, we have a separate file for routes called `routes.rb`  

**Model**  
The model represents the information and the data from the database. It is as independent from the database as possible (Rails comes with its own O/R-Mapper, allowing you to change the database that feeds the application but not the application itself). The model also does the validation of the data before it gets into the database. Most of the time you will find a table in the database and an according model in your application.  
  
**View**  
The view is the presentation layer for your application. The view layer is responsible for rendering your models into one or more formats, such as XHTML, XML, or even Javascript. Rails supports arbitrary text rendering and thus all text formats, but also includes explicit support for Javascript and XML. Inside the view you will find (most of the time) HTML with embedded Ruby code. In Rails, views are implemented using ERb by default.  
  
**Controller**  
The controller connects the model with the view. In Rails, controllers are implemented as ActionController classes. The controller knows how to process the data that comes from the model and how to pass it onto the view. The controller should not include any database related actions (such as modifying data before it gets saved inside the database). This should be handled in the proper model.  
  
**Helper**  
When you have code that you use frequently in your views or that is too big/messy to put inside of a view, you can define a method for it inside of a helper. All methods defined in the helpers are automatically usable in the views.  
  
ref: [Rails Wiki](http://en.wikibooks.org/wiki/Ruby_on_Rails/Getting_Started/Model-View-Controller)  
  
## Convention over Configuration  
Rails goes with the motto "Convention over Configuration". So, instead of having to spend a lot of time configuring options, we follow a set of conventions.  
For example, in Sinatra, we might have something like  
```ruby
# app.rb
get "/" do
  @task = Task.all
  erb :index          # Here we have to state erb :index to render this view
end  

# Rails 
def index             # In Rails, we just define a method for each view in its controller
end
```  
  
## Gemfile & Bundler
Our Gemfile stores all the gems we use in our application.  
```ruby
# Gemfile

source 'https://rubygems.org'         # this is currently (and for the foreseeable future) the main source for rubygems

gem 'rails', '4.0.2'                  # this uses the specfic rails version '4.0.2'
gem 'pg'
gem 'sass-rails', '~> 4.0.0'          # this will use sass-rails up to version 4.0.9
gem 'uglifier', '>= 1.3.0'            # this will use an uglifier version greater than or equal to 1.3.0
gem 'coffee-rails', '~> 4.0.0'
gem 'jquery-rails'
gem 'turbolinks'
gem 'jbuilder', '~> 1.2'

group :doc do
  # bundle exec rake doc:rails generates the API under doc/api.
  gem 'sdoc', require: false
end

```  
Add thin to your Gemfile  
```ruby
# Gemfile

source 'https://rubygems.org'         # this is currently (and for the foreseeable future) the main source for rubygems

gem 'rails', '4.0.2'                  # this uses the specfic rails version '4.0.2'
gem 'pg'
gem 'thin'                            # add thin instead of webrick

gem 'sass-rails', '~> 4.0.0'          # this will use sass-rails up to version 4.0.9
gem 'uglifier', '>= 1.3.0'            # this will use an uglifier version greater than or equal to 1.3.0
gem 'coffee-rails', '~> 4.0.0'
gem 'jquery-rails'
gem 'turbolinks'
gem 'jbuilder', '~> 1.2'

group :doc do
  # bundle exec rake doc:rails generates the API under doc/api.
  gem 'sdoc', require: false
end

group :development, :test do          # require gems for development and test environments
  gem 'debugger'
  gem 'rspec-rails'
end

```  
Then run `bundle install` to update your app to use thin, and make sure to restart your server  
```bash
ctrl-c
rails s
```  
## REST  
Routes in rails use [RESTful architecture](http://en.wikipedia.org/wiki/Representational_state_transfer} (Representational state transfer). Let's look at what this means.  
  
Inside your app's directory, in the terminal type `rails generate controller home`. Then, open up your routes.rb file.  
```ruby
# config/routes.rb
AwesomeAnswers::Application.routes.draw do

  get "/about_us" => "home#about"         # we add get, then give a path, followed a hashrocket. We then reference
                                          # 'home' which is a controller, and about (method in the home controller)
end

```  
get is an HTTP verb.
  * GET
  * POST
  * PATCH/PUT
  * DELETE  
  
Add a method to your home_controller called about.  

```ruby
# app/controllers/home_controller.rb
class HomeController < ApplicationController

  def about
    render text: "Welcome"
  end
  
end
```  
Here we are just rendering the text "Welcome". However, by default, the method name in the controller will look for a view.erb file. So, let's set one up. Create a page called about.erb in your app/views/[controller] directory.  
```ruby
# app/views/home/about.erb

<h1>Hello!</h1>

```  
Now, remove the line `render text: "Welcome"` from your home_controller.  
```ruby
# app/controllers/home_controller.rb
class HomeController < ApplicationController

  def about
  end
  
end
```  
Let's add an FAQ.  
```ruby
# config/routes.rb
get "/faq" => "home#faq"          # add this line to your routes file
```  
define a method in your home controller.  
```ruby
# app/controllers/home_controller.rb
class HomeController < ApplicationController

  def about
  end
  
  def faq
  end
  
end
```  
Create an faq.erb file in your views directory  
```ruby
# app/views/home/faq.erb
<h1>FAQ</h1>
```  
What's a controller?  
It's a class  
```ruby
app/controllers/home_controller.rb
class HomeController < ApplicationController              # our home contoller inherits from ApplicationController.

  def about
  end
  
  def faq
  end

end
```  
application_controller.rb
```ruby
app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception
end
```  
`protect_from_forgery` makes it so you can't easily put something or post something, without an authorization token. Basically by having this in the application_controller, all my controllers have this as long as they *inherit from this controller*.  
  
```ruby
# config/environments/production.rb
AwesomeAnswers::Application.configure do

  config.cache_classes = true

  config.eager_load = true

  # Full error reports are disabled and caching is turned on.
  config.consider_all_requests_local       = false
  config.action_controller.perform_caching = true

  # Enable Rack::Cache to put a simple HTTP cache in front of your application
  # Add `rack-cache` to your Gemfile before enabling this.
  # For large-scale production use, consider using a caching reverse proxy like nginx, varnish or squid.
  # config.action_dispatch.rack_cache = true

  # Disable Rails's static asset server (Apache or nginx will already do this).
  config.serve_static_assets = false

  # Compress JavaScripts and CSS.
  config.assets.js_compressor = :uglifier
  # config.assets.css_compressor = :sass

  # Do not fallback to assets pipeline if a precompiled asset is missed.
  config.assets.compile = false

  # Generate digests for assets URLs.
  config.assets.digest = true

  # Version of your assets, change this if you want to expire all your assets.
  config.assets.version = '1.0'

  # Force all access to the app over SSL, use Strict-Transport-Security, and use secure cookies.
  # config.force_ssl = true  # Here we can force_ssl

  # Set to :debug to see everything in the log.
  config.log_level = :info

  # Prepend all log lines with the following tags.
  # config.log_tags = [ :subdomain, :uuid ]

  # Use a different logger for distributed setups.
  # config.logger = ActiveSupport::TaggedLogging.new(SyslogLogger.new)

  # Use a different cache store in production.
  # config.cache_store = :mem_cache_store

  # Enable serving of images, stylesheets, and JavaScripts from an asset server.
  # config.action_controller.asset_host = "http://assets.example.com"

  # Precompile additional assets.
  # application.js, application.css, and all non-JS/CSS in app/assets folder are already added.
  # config.assets.precompile += %w( search.js )

  # Ignore bad email addresses and do not raise email delivery errors.
  # Set this to true and configure the email server for immediate delivery to raise delivery errors.
  # config.action_mailer.raise_delivery_errors = false

  # Enable locale fallbacks for I18n (makes lookups for any locale fall back to
  # the I18n.default_locale when a translation can not be found).
  config.i18n.fallbacks = true

  # Send deprecation notices to registered listeners.
  config.active_support.deprecation = :notify

  # Disable automatic flushing of the log to improve performance.
  # config.autoflush_log = false

  # Use default logging formatter so that PID and timestamp are not suppressed.
  config.log_formatter = ::Logger::Formatter.new
end
```  
If I want to create a special section in my website for help, what should I do? Where should I start? (in terminal in the directory of your  application)
```bash
rails generate controller help  

#############################
#### To see the options available to generate, try just rails generate
#############################

rails generate

Rails:
  assets
  controller
  generator
  helper
  integration_test
  jbuilder
  mailer
  migration
  model
  resource
  scaffold
  scaffold_controller
  task

Coffee:
  coffee:assets

Jquery:
  jquery:install

Js:
  js:assets

TestUnit:
  test_unit:plugin

```   

Then add some routes to the routes.rb  
```ruby
# config/routes.rb
# ...
  get "/help" => "help#index"
  
#...
```  

Add an index method to the help controller
```ruby
class HelpController < ApplicationContoller
  
  def index
  end

end  
```  
  
Add an index.erb inside a help directory to the views diretory  
```ruby
app/views/help/index.erb
<h1>Welcome to the help section</h1>
```   
  
We can access all the routes available in our app if we go to [localhost:3000/rails/info/routes](http://localhost:3000/rails/info/routes)  
  
We can see in our routes that rails automatically generates 'helpers' for us. Rather than /about_us, we now have a rails method we can use to access this route through our app called `about_us_path`.  
  
We use this to create links, for example add a navigation section
```ruby
# app/views/layouts/application.html.erb
<!DOCTYPE html>
<html>
<head>
  <title>AwesomeAnswers</title>
  <%= stylesheet_link_tag    "application", media: "all", "data-turbolinks-track" => true %>
  <%= javascript_include_tag "application", "data-turbolinks-track" => true %>
  <%= csrf_meta_tags %>
</head>
<body>

# Add a navigation section here

  <%= link_to "About Us", about_us_path, class: "btn btn-primary", id: "about" %> | 
  <%= link_to "FAQ", faq_path %> | 
  <%= link_to "Help", help_path %>

<%= yield %>

</body>
</html>

```  
## Rails Resources  

If I have a resource called post, it will be a model Post.rb, and a controller posts_controller.rb. Models are given singular names, and controllers are given the plural of the model, by convention, and this is how rails works.    
Let's start by creating a controller: `rails generate controller questions`.  
  
To show all the questions, we can define a method called index in the questions_controller.  
```ruby
class QuestionsController < ApplicationController

  def index
  end
  
end
```  
And add a route 
```ruby
# config/routes.rb
AwesomeAnswers::Application.routes.draw do

  get "/" => "home#index"

  get "/about_us" => "home#about"

  get "/faq" => "home#faq"

  get "/help" => "help#index"

  get "/questions" => "questions#index"

end
```  
Add an index.html.erb page to the view  
```ruby
# app/views/qustion/index.html.erb
<h1>Listing All Questions</h1>


```  
If I want to create a question, I need to define a method create. What should the route for this be? `post "/questions => "questions#index"` 
```ruby
# config/routes.rb
AwesomeAnswers::Application.routes.draw do

  get "/" => "home#index"

  get "/about_us" => "home#about"

  get "/faq" => "home#faq"

  get "/help" => "help#index"

  get "/questions" => "questions#index"
  post "/questions" => "questions#index"

end
```  
If  want to show a specific question, what should I do? That's right! `get "/questions/:id" => "questions#show"`  
```ruby
# config/routes.rb
AwesomeAnswers::Application.routes.draw do

  get "/" => "home#index"

  get "/about_us" => "home#about"

  get "/faq" => "home#faq"

  get "/help" => "help#index"

  get "/questions" => "questions#index"
  post "/questions => "questions#index"
  get "/questions/:id" => "questions#show"

end
```   
And of course, then define the `show` method in the questions_controller.  
```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController

  def index
  end

  def create
  end
  
  def show
    render text: "The id is: #{params[:id]}"     #we can get the question based on it's ID. This is available through params
  end

end
```  
To edit, update, and destroy a question, we can add the according methods and routes.  
```ruby
# config/routes.rb
AwesomeAnswers::Application.routes.draw do

  root "questions#index"        # specify the root path just like
                                # get "/" => "questions#index"

  get "/about_us" => "home#about"

  get "/faq" => "home#faq"

  get "/help" => "help#index"

  get "/questions"          => "questions#index"
  post "/questions"         => "questions#create"
  get "/questions/:id"      => "questions#show"
  get "/questions/:id/edit" => "questions#edit"
  match "/questions/:id"    => "questions#update", via: [:put, :patch]
  delete "/questions/:id"   => "questions#destroy"
  
  
  ### Note, we can create all these routes by adding the simple line
  resources :questions
  
  ### to limit the routes available to questions, we can add only
  resources :questions, only: [:index, :new, :create]

end
```   

Methods
```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController

  def index
  end

  def create
    render text: "Create a question"
  end

  def show
    render text: "The id is: #{params[:id]}"
  end

  def edit
    render text: "Editing question: #{params[:id]}"
  end

  def new
    render text: "A new question"
  end

end
```  
If I want to do something like vote on a question, I can add a path to the routes.rb such as:  
```ruby
# config/routes.rb
  resources :questions do 
    post :vote_up, on: :member
  end
```  
  
This will create the route `vote_up_question_path  POST  /questions/:id/vote_up(.:format)  questions#vote_up`. We are voting up on member, because we are selecting a particular member in a collection. If want to search a collection of questions, I would add `post :search, on: :collection` to the routes resource. A collection doesn't require an id, whereas a member does.  
```ruby
# config/routes.rb
  resources :questions do 
    post :vote_up, on: :member
    post :search, on: :collection
  end
```   
This creates the route `search_questions_path  POST  /questions/search(.:format)   questions#search`.  If we want to have a series of methods on a member or collection, he's the syncax.  
```ruby
# config/routes.rb
  resources :questions, only: [:index, :create, :show] do     # in contrast to only, we could use except: [:update, :create], etc.
    member do 
      post :vote_up
      post :vote_down
    end
    post :search, on: :collection
  end
```