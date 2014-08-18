# Form Objects  
let's take a look at [simple form](https://github.com/plataformatec/simple_form).  
Add `gem 'simple_form'` to your Gemfile, run `bundle install`, and if you're using bootstrap, run `rails generate simple_form:install --bootstrap`. Otherwise just `rails generate simple_form:install`.  
  
This will give the following output, which is rather meaningful (read it).  
```bash
  Be sure to have a copy of the Bootstrap stylesheet available on your
  application, you can get it on http://twitter.github.com/bootstrap.

  Inside your views, use the 'simple_form_for' with one of the Bootstrap form
  classes, '.form-horizontal', '.form-inline', '.form-search' or
  '.form-vertical', as the following:

    = simple_form_for(@user, html: {class: 'form-horizontal' }) do |form|
```
So, if we want to actually apply this to say our questions form, we can open it up, change `form_for` to `simple_form_for`, and add a class such as 'form-horizontal'. Then you can totally remove all the classes and labels and greatly simplify your form code.
```ruby
# app/views/questions/_form.html.haml

- if @question.errors.any?
  %ul
    - @question.errors.full_messages.each do |message|
      %li message

= simple_form_for @question, html: {class: "form-horizontal"}  do |f|
  = f.input :title, label: "question title", input_html: {class: "form-control"}
  = f.input :description, label: "questions description", input_html: {class: "form-control"}
  = f.collection_check_boxes :category_ids, Category.order("title"), :id, :title
  %br
  = f.submit class: "btn btn-default"
```
If you want to simplify this, we can use the edge version of simple form. To do this, add the github repo address to the gemfile
```ruby
# Gemfile

# ...

gem 'simple_form', git: 'https://github.com/plataformatec/simple_form.git'

# ...
```

Then detroy the install
```bash
rails destroy simple_form:install
```
And re-install 
```bash
rails generate simple_form:install --bootstrap
```
If you get this in terminal:
```bash
Overwrite /Users/ogryzek/CodeCore/cohort2/ruby/awesome_answers/config/initializers/simple_form_bootstrap.rb? (enter "h" for help) [Ynaqdh]
```
Type `Y` and hit enter. This should output something like
```bash
       force  config/initializers/simple_form_bootstrap.rb
       exist  config/locales
      create  config/locales/simple_form.en.yml
      create  lib/templates/haml/scaffold/_form.html.haml
===============================================================================

  Be sure to have a copy of the Bootstrap stylesheet available on your
  application, you can get it on http://getbootstrap.com/.

  Inside your views, use the 'simple_form_for' with one of the Bootstrap form
  classes, '.form-horizontal' or '.form-inline', as the following:

    = simple_form_for(@user, html: { class: 'form-horizontal' }) do |form|

===============================================================================
```
And all is well.  
  
Now, we can remove the excess html from the forms, and it will automatically assign the correct classes of form-control where needed. It will also give us color highlighting when required fields are not filled out.  
```ruby
# app/views/questions/_form.html.haml

- if @question.errors.any?
  %ul
    - @question.errors.full_messages.each do |message|
      %li message


= simple_form_for @question, html: {class: "form-horizontal"}  do |f|
  = f.input :title, label: "question title"
  = f.input :description, label: "questions description"
  = f.collection_check_boxes :category_ids, Category.order("title"), :id, :title, item_wrapper_class: "form-group"
  /= f.collection_check_boxes :category_ids, Category.order("title"), :id, :title
  = f.submit class: "btn btn-default"
```
**note**: You can still chain classes by adding them as we did previously