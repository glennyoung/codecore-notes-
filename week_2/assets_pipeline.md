# Assets Pipeline in Rails  

  * app
    * assets
      * images
      * javascripts
      * stylesheets  
Rails uses a gem called [sprockets](https://github.com/sstephenson/sprockets) to handle the assets pipeline. This gives us some [directives](https://github.com/sstephenson/sprockets#sprockets-directives) available to require and access different files and folders within our assets.  
  
***Sprockets Directives*** 
```bash
require
include
require_directory
The require Directive
require_tree
require_self
depend_on
The depend_on_asset
stub
```
  
application.css
```css
# app/assets/stylesheets/application.css
/*
 * This is a manifest file that'll be compiled into application.css, which will include all the files
 * listed below.
 *
 * Any CSS and SCSS file within this directory, lib/assets/stylesheets, vendor/assets/stylesheets,
 * or vendor/assets/stylesheets of plugins, if any, can be referenced here using a relative path.
 *
 * You're free to add application-wide styles to this file and they'll appear at the top of the
 * compiled file, but it's generally better to create a new file per style scope.
 *
 *= require_self
 *= require_tree .
 *= require_directory ./abc
 */
```  
## Layouts 
**nifty gem**: [rails-layouts](http://railsapps.github.io/rails-default-application-layout.html)  
**note**: `%w` is shorthand for an array.  
```bash
my_array = %w(a b c d e)
# is equivalent to
my_array = ["a", "b", "c", "d", "e"]
```  
If you want different styles for different pages or namespaces on your site, you can add layouts.  

Uncomment line 62 in your config.ru, and add a reference to external.css  
```ruby
# config.ru line 62
config.assets.precompile += %w( external.css )
```  
## Rake Tasks  
To see a list of tasks rake comes with try `bundle exec rake -T`. This will give you something like this  
```bash
rake about                              # List versions of all Rails frameworks and the environment
rake assets:clean[keep]                 # Remove old compiled assets
rake assets:clobber                     # Remove compiled assets
rake assets:environment                 # Load asset compile environment
rake assets:precompile                  # Compile all the assets named in config.assets.precompile
rake cache_digests:dependencies         # Lookup first-level dependencies for TEMPLATE (like messages/show or ...
rake cache_digests:nested_dependencies  # Lookup nested dependencies for TEMPLATE (like messages/show or comme...
rake db:create                          # Create the database from DATABASE_URL or config/database.yml for the...
rake db:drop                            # Drops the database using DATABASE_URL or the current Rails.env (use ...
rake db:fixtures:load                   # Load fixtures into the current environment's database
rake db:migrate                         # Migrate the database (options: VERSION=x, VERBOSE=false, SCOPE=blog)
rake db:migrate:status                  # Display status of migrations
rake db:rollback                        # Rolls the schema back to the previous version (specify steps w/ STEP=n)
rake db:schema:cache:clear              # Clear a db/schema_cache.dump file
rake db:schema:cache:dump               # Create a db/schema_cache.dump file
rake db:schema:dump                     # Create a db/schema.rb file that can be portably used against any DB ...
rake db:schema:load                     # Load a schema.rb file into the database
rake db:seed                            # Load the seed data from db/seeds.rb
rake db:setup                           # Create the database, load the schema, and initialize with the seed d...
rake db:structure:dump                  # Dump the database structure to db/structure.sql
rake db:version                         # Retrieves the current schema version number
rake doc:app                            # Generate docs for the app -- also available doc:rails, doc:guides (o...
rake log:clear                          # Truncates all *.log files in log/ to zero bytes (specify which logs ...
rake middleware                         # Prints out your Rack middleware stack
rake notes                              # Enumerate all annotations (use notes:optimize, :fixme, :todo for focus)
rake notes:custom                       # Enumerate a custom annotation, specify with ANNOTATION=CUSTOM
rake rails:template                     # Applies the template supplied by LOCATION=(/path/to/template) or URL
rake rails:update                       # Update configs and some other initially generated files (or use just...
rake routes                             # Print out all defined routes in match order, with names
rake secret                             # Generate a cryptographically secure secret key (this is typically us...
rake stats                              # Report code statistics (KLOCs, etc) from the application
rake test                               # Runs test:units, test:functionals, test:integration together
rake test:all                           # Run tests quickly by merging all types and not resetting db
rake test:all:db                        # Run tests quickly, but also reset db
rake test:recent                        # Run tests for {:recent=>["test:deprecated", "test:prepare"]} / Depre...
rake test:uncommitted                   # Run tests for {:uncommitted=>["test:deprecated", "test:prepare"]} / ...
rake time:zones:all                     # Displays all time zones, also available: time:zones:us, time:zones:l...
rake tmp:clear                          # Clear session, cache, and socket files from tmp/ (narrow w/ tmp:sess...
rake tmp:create                         # Creates tmp directories for sessions, cache, sockets, and pids
```  
If you want to precompile your assets to see what they look like, you can run  
```bash
bundle exec rake assets:precompile RAILS_ENV=production
```  
***Note***: You may need to add a production database to your database.yml. In this case, we're using our test database for production locally.    
```yml
development:
  adapter: postgresql
  encoding: unicode
  database: awesome_answers_development
  pool: 5
  username: [my-user-name]
  password:

test:
  adapter: postgresql
  encoding: unicode
  database: awesome_answers_test
  pool: 5
  username: [my-user-name]
  password:

production:
  adapter: postgresql
  encoding: unicode
  database: awesome_answers_test
  pool: 5
  username: [my-user-name]
  password:

```
```bash
bundle exec rake assets:precompile
```
This will give you an assets directory in your public folder.  

To add an image, simply add one to your images directory in the assets path, then you can access it with image_tag    
```
# app/views/question/index.html.erb
<h1>Welcome to our Site</h1>

<%= image_tag "drewbro.jpg" %>
```  
## Add Bootstrap to your App  
In your Gemfile add the bootstrap gem  
```ruby
# Gemfile

source 'https://rubygems.org'

gem 'rails', '4.0.2'
gem 'pg'
gem 'thin'
gem 'bootstrap-sass', '~> 3.1.1.0'


gem 'sass-rails', '~> 4.0.0'
gem 'uglifier', '>= 1.3.0'
gem 'coffee-rails', '~> 4.0.0'
gem 'jquery-rails'
gem 'turbolinks'
gem 'jbuilder', '~> 1.2'

group :development do
  gem 'hirb'
  gem 'interactive_editor'
  gem 'awesome_print'
  gem 'quiet_assets'
end

group :doc do
  gem 'sdoc', require: false
end
```  
Then run `bundle install` in the terminal.  
  
in app/assets/stylesheets add a file like `bootstrap_and_css_overrides.css.scss`
```css
# app/assets/stylesheets/bootstrap_and_css_overrides.css.scss
@import 'bootstrap';
```  

## Deploy to Heroku  
Because we're deploying to heroku, if you have done a `rake assets:precompile`, you should delete your assets directory in the public folder.  
```ruby
git init
git add .
git commit -m "Initial commit"
git log

heroku create
git remote -v
git push heroku master

heroku run rake db:migrate

heroku open


```