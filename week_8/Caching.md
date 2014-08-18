# Caching

We can look into optimizing queries. We can simplify the query, optimize it, etc. Generally the gain from this isn't really all that high. We may notice that of something like 55ms of load time, 51.9ms will be in rendering. That's why from image caching, we can get bigger gains than from other things.  
  
We're going to look at fragment caching. Open up development.rb and production.rb. We can see it's disabled in dev and enabled in prod. We're going to turn in on for our development so we can test it.

```ruby
# config/environments/development.rb 

config.action_controller.perform_caching = true
```
Then restart your server.  
  
If you look on your homepage, you'll see all we're doing is looking for questions and looping through them one by one. So go to the questions controller and add another instance variable called recent questions.
```ruby
# app/controllers/questions_controller.rb

# ...

  def index
    @recent_questions = Question.recent(3)
    @questions = Question.all
  end

# ...

```
Then on the view, copy out the part that's outputting the questions, but instead if using questions, use recent questions!
```haml
/ app/views/questions/index.html.haml

%h2 Recent Questions
.col-md-6.col-md-offset-3
  - cache "recent questions" do
    - @recent_questions.each do |question|
      = cache ["recent", question] do
        .droppy.well.col-md-6.questions{:id =>"recent_#{dom_id(question)}"}
          %h2= link_to question.title, question_path(question)
          %p= question.description
          %p
            Created On: #{formatted_date(question.created_at)}

%h2 All Questions
.col-md-6.col-md-offset-3
  - @questions.each do |question|
    = cache question do
      .droppy.well.col-md-6.questions{:class => dom_id(question)}
        %h2= link_to question.title, question_path(question)
        %p= question.description
        %p
          Created On: #{formatted_date(question.created_at)}
```