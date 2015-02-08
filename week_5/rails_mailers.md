# Rails Mailers  
[Mandrill](http://mandrill.com/) | 
In rails, malers are built into the framework. If you go to Gemfile.lock, you can see the gem actionmailer. In the app folder, there's a mailer folder right there. It's empty by default. So, the first step is to set up the mailer.  
  
Let's create a file called initializers/setup_mail.rb, and put the setup information inside it.  
```ruby
# initializers/setup_mail.rb

ActionMailer::Base.smtp_settings = {
  address:                'smtp.gmail.com',
  port:                   '587',
  enable_starttls_auto:   true,
  user_name:              ENV['email_username'],
  password:               ENV['email_password'],
  authentication:         :plain
}
```  
In `.gitignore`, make sure to ignore the email variables, `/config/initializers/email_vars.rb`  
```.gitignore
# config/initializers/email_vars.rb

#...

/config/initializers/email_vars.rb

#...
```  
Now, create the file email_vars.rb  
```ruby
# config/initializers/email_vars.rb
# this is one of Tam's gmail accounts

ENV['email_username'] = 'answerawesome'
ENV['email_password'] = 'Sup3rSecret'
```  
Let's generate a new model called answer mailer:  `rails generate mailer answer_mailer`  
  
```ruby
# app/mailers/answer_mailer.rb
class AnswerMailer < ActionMailer::Base
  default from: "from@example.com"

  def notify_question_owner(answer)
  end
  
end
```  
Create two view files in app/views/answer_mailer  
```haml
# app/views/answer_mailer/notify_question_owner.txt.haml  
Hello #{@reciever.full_name},

You've got an answer for your question: 

Question: #{@question.title}

Answer: #{@answer.body}

Regards,

Awesome Answers Team
```  
And one that is html/haml  
```haml
# app/views/answer_mailer/notify_question_owner.html.haml 
%p Hello #{@reciever.full_name},
%br 
%p You've got an answer for your question: 
%br 
%p Question: #{@question.title}
%br 
%p Answer: #{@answer.body}
%br
%p Regards,
```  
Then in the answers controller, modify the create to send an email  
```ruby
# app/controllers/answers_controller.rb
 
 #...
 
  def create
    @question = Question.find params[:question_id]
    @answer = @question.answers.new(answer_attributes)
    @answer.user = current_user
    if @answer.save
      # Answer.mailer.notify_question_owner(@answer).deliver
      AnswerMailer.notify_question_owner(@answer).deliver
      redirect_to @question, notice: "Answer created successfully."
    else
      render "/questions/show"
    end
  end
  
  #...
```
Add the letter opener gem to your development group in the Gemfile
```ruby
group :development do
  gem 'letter_opener'
  gem 'hirb'
  gem 'interactive_editor'
  gem 'awesome_print'
  gem 'quiet_assets'
  gem 'binding_of_caller'
  gem 'better_errors'
  gem 'faker'
end
```  
Run bundle, then add this line to your development.rb
```ruby
# config/environments/development.rb

config.action_mailer.delivery_method = :letter_opener

```  
This will make the email open in a new browser window for your development environment.  
  
## Background Jobs  
[resque](https://github.com/resque/resque) |  
  
Delayed job with a normal database is more than enough for regular background jobs. However, if you have a lot going on, you may wish to checkout resque.  
  
Open up your Gemfile and add the gem [delayed_job_active_record](https://github.com/collectiveidea/delayed_job_active_record). Then in the terminal run, `rails g delayed_job:active_record` then `rake db:migrate`  
  
In the answers controller modify the AnswerMailer line  
```ruby
# app/controllers/answers_controller.rb

# from
Answer.mailer.notify_question_owner(@answer).deliver

# to
AnswerMailer.delay.notify_question_owner(@answer)

```  
Add these two gems to the Gemfile  
```ruby
# Gemfile.rb

#...

gem 'delayed_job_active_record'
gem 'delayed_web_job'

#...

```

```bash
# in rails c check for your delayed jobs
Delayed::Job.all

# in the terminal run the delayed jobs
rake jobs:work
```  
Add the [delayed job web gem](https://github.com/collectiveidea/delayed_job)  
  
## Rake Tasks  
  
Make sure you have the faker gem in your gemfile `gem 'faker', group: [:development, :test]`
  
In the command line type `rake -T` to view a list of tasks. Now, let's generate our own task. In the command line type `rails generate task random_question_generator` Then open up the task file.  
```rake
# lib/tasks/random_question_generator.rake

namespace :generator do

  desc "Generate ten questions with 10 answers each"
  task :questions_and_answers => :environment do

    10.times do
      question = Question.create(title: Faker::Lorem.sentence(10), description: Faker::Lorem.sentence(30))
      10.times do
        question.answers.create(body: Faker::Company.bs)
      end
    end

  end

end
```  
Then to run the generator, in the command line enter: `rake generator:questions_and_answers`  
  
```
class Device
  def deliver
    # long runny method
  end
  handle asynchronously :deliver
end

device = Devise.new
device.deliver
```


In order for the mailer templates to work, two more files need to be added to app/views/layouts/ called mailer.text.erb and mailer.html.erb

content for mailer.html.erb 
  html
    body
      <%= yield %>
    /body>
  /html
  
content for mailer.text.erb 
  <%= yield %>

regards, 
  zeus-
