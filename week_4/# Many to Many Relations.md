# Many to Many Relations  
Start by creating category and categorization models.  
```bash
rails generate model category title:string
rails generate model categorization category:references question:references
bundle exec rake db:migrate
```  
Open up the rails console, and let's come up with an array of categories then create them.  
```bash
# Add the categories
["Art", "Science", "Technology", "Sports", "Travel", "Humor"].each {|x| Category.create(title: x)}
# or
%w(Art Science Technology Sports Travel Humor).each {|x| Category.create(title: x)}

# View the categories
Category.all

# Somecool shorthand with Ampersand and map
Category.all.map(&:title)
```  

Open up Category and add has many associations
```ruby
# app/models/category.rb
class Category < ActiveRecord::Base
  has_many :categorizations, dependent: :destroy
  has_many :questions, through: :categorizations
end
```  
Add associations to the answer model
```ruby
# app/models/question.rb
class Question < ActiveRecord::Base
#...
  has_many :categorizations, dependent: :destroy
  has_many :categories, through: :catorizations
#...
end

```  
More rails console stuff  
```bash
q = Question.first
c1 = Category.first
c1 = Category.find 2
q.categories << c1
q.categories << c2
q.categories
Categorization.all
```  
Add display of errors to questions form
```haml
- if @question.errors.any?
  %ul
    - @question.errors.full_messages.each do |message|
      %li message

= form_for @question do |f|
  .form-field
    = f.label :title, "Title"
    = f.text_field :title, class: "form-control"
  .form-field
    = f.label :description
    = f.text_area :description, class: "form-control"
  .form-field
    = check_box_tag(:pet_dog)
    = label_tag(:pet_dog, "I own a dog")
    = check_box_tag(:pet_cat)
    = label_tag(:pet_cat, "I own a cat")
  = f.submit class: "btn btn-default"
```
  
Then in the questions controller, add category ids to the permited params in the private question attributes method.  
```ruby
#...
  def question_attributes
    params.require(:question).permit([:title, :description, {category_ids: []}])
  end
#...
```  
Add some checkboxes for your question form  
```haml
/ app/views/questions/_form.html.haml
- if @question.errors.any?
  %ul
    - @question.errors.full_messages.each do |message|
      %li message

= form_for @question do |f|
  .form-field
    = f.label :title, "Title"
    = f.text_field :title, class: "form-control"
  .form-field
    = f.label :description
    = f.text_area :description, class: "form-control"
  
  = f.collection_check_boxes :category_ids, Category.order("title"), :id, :title
  %br
  = f.submit class: "btn btn-default"
```
Add the categories to the question show page  
```haml
%h1= @question.title
%p
  = @question.description
  %p
    = formatted_date(@question.created_at)
    = succeed "Vote" do
      %br
    Count: #{@question.vote_count}
    %br
    - if @question.categories.present?
      Categories (#{@question.categories.count}):
      = @question.categories.map(&:title).join(", ")
    %br
    - if session[:has_voted]
      You voted already!
    - else
      = button_to "Vote Up", vote_up_question_path(@question)
    %br/
    = button_to "Delete", @question, method: :delete, data: { confirm: "Are you sure you want to delete this question?" }, class: "btn btn-default"
    %br/
    = link_to "All Questions", questions_path

    %h2 Add an Answer
    - if @answer.errors.any?
      %ul
        - @answer.errors.full_messages.each do |message|
          %li= message

    = form_for @answer, url: question_answers_path(@question) do |f|
      = f.text_area :body, class: "form-control"
      %br
      = f.submit "Submit an answer", class: "btn btn-primary"

    %hr
    -@question.answers.each do |answer|
      = render "/answers/answer", answer: answer
```
