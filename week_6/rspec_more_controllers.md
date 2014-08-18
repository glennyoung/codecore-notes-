# RSpec: More Controllers

Let's make an answers controller spec
```ruby
# spec/controllers/answers_controller_spec.rb
require 'spec_helper'

describe AnswersController do

  let(:user) { create(:user) }
  let(:question) { create(:question) }

  context "user signed in" do
    before {sign_in user}
    describe "#create" do

      def valid_request
        post :create, question_id: question.id,
                        answer: {body: "Valid body"}
      end

      it "adds an answer record to the database" do
        expect do
          valid_request
        end.to change { question.answers.count }.by(1)
      end

      it "sends an email to question owner" do
        ActionMailer::Base.deliveries.clear
        expect.(ActionMailer::Base.deliveries).to have(1).item
      end
    end
  end

end
```
Now, let's make a spec to test our answer mailer. We can make a folder in the spec directory called mailers, and a file inside that.
```ruby
# spec/mailers/answer_mailer_spec.rb

require 'spec_helper'

describe AnswerMailer do

  let(:user) { create(:user)}
  let(:user1) { create(:user) }
  let(:question) { create(:question, user: user)}
  let(:answer) { create(:answer, question: question, user: user1)}

  describe "#notify_question_owner" do

    before do
      @mail = AnswerMailer.notify_question_owner(answer)
    end

    it "sends email to question owner" do
      @mail = AnswerMailer.notify_question_owner(answer)
      expect(@mail.to).to eq([user.email])
    end

    it "sends email from noreply@awesomeanswers.com" do
      mail = AnswerMailer.notify_question_owner(answer)
      expect(@mail.from).to eq(["noreply@awesomeanswers.com"])
    end

    it "contains the answer body in the email body" do
      puts ">>>>>>>>>>>>>>>>> #{@mail.body.class}" # testing something.
      expect(@mail.body.to_s).to match /#{answer.body}/i
    end
  end


end
```
Add a user association to the answers factory.  
```ruby
# spec/factories/answers.rb

FactoryGirl.define do
  factory :answer do
    association :user, factory: :user
    association :question, factory: :question
    body Faker::Company.bs
  end
end
```
Change the email address in your answers mailer to pass the tests
```ruby
# app/mailers/answer_mailer.rb

class AnswerMailer < ActionMailer::Base
  default from: "noreply@awesomeanswers.com"

  def notify_question_owner(answer)
    @answer = answer
    @question = answer.question
    @receiver = @question.user
    mail(to: @receiver.email, subject: "You've got new answer on your question.")
  end

end
```
We can refactor our answers mailer spec now
```ruby
# spec/mailers/answer_mailer_spec.rb
require 'spec_helper'

describe AnswerMailer do

  let(:user) { create(:user)}
  let(:user1) { create(:user) }
  let(:question) { create(:question, user: user)}
  let(:answer) { create(:answer, question: question, user: user1)}

  describe "#notify_question_owner" do

    subject { AnswerMailer.notify_question_owner(answer) }

    its(:to) { should eq([user.email]) }
    its(:from) { should eq(["noreply@awesomeanswers.com"]) }
    its("body.to_s") { should match /#{answer.body}/i }

  end


end
```
Add the gem [simplecov](https://github.com/colszowka/simplecov) to your Gemfile
```ruby
# Gemfile.rb

# ...

  gem 'simplecov', require: false, group: :test

# ...
```
Then open up your spec helper and add the following code to the very top.  
```ruby
# spec/spec_helper.rb
require 'simplecov'
SimpleCov.start

# ...
```
Do a `bundle install` and run your specs. The last line will give you an address that you can pop into your browser and see some great information.  
  
## Testing Views
Let's create a new folder in our spec directory called views, and in that folder a new folder called questions. Inside there, let's create a spec to test new.  
```ruby
# spec/views/questions/new.html.haml_spec.rb
require 'spec_helper'

describe "questions/new.html.haml" do

  before do
    assign(:question, stub_model(Question))
    render
  end

  it "contains text 'Create New Question'" do
    expect(rendered).to match /Create New Question/i
  end

  it "renders '_form' template" do
    expect(rendered).to render_template(partial: "_form")
  end

end
```  
Let's do some real views testing with CapyBara. Create another folder in your spec called features, and a file for the home page spec
```ruby
spec/features/home_page_spec.rb
require 'spec_helper'

feature "Visiting Home Page" do

  it "contains the text 'home'" do
    visit root_path  # visit is a capabara thing
    expect(page).to have_text /home/i     # page is also a capabara thing
  end
end
```
Now, let's add another feature spec for the questions listings page.  
```ruby
# spec/features/listing_questions.rb

require 'spec_helper'

feature "Listing Questions Page" do
  it "should list questions in the database" do
    question = create(:question)
    visit questions_path
    expect(page).to have_text /#{question.title}/i
  end
end
```
Add another feature spec for creating a question.  
```ruby
# spec/features/question_create.rb
require 'spec_helper'

feature "Creating A Question" do

  it "creates an answer in the database" do
    visit questions_path
    click_on "Create a Question"
    save_and_open_page
  end
end
```
Check out [how to test with devise and capybara](https://github.com/plataformatec/devise/wiki/How-To:-Test-with-Capybara) to get the following.
```ruby
include Warden::Test::Helpers
Warden.test_mode!
```
Now let's add some more tests!
```ruby
# spec/features/question_create.rb

require 'spec_helper'

feature "Creating A Question" do
  let(:user) { create(:user) }
  let(:question) { create(:question, user: user) }

  before do
    login_as(user, :scope => :user)
  end

  it "creates an answer in the database" do
    visit questions_path
    click_on "New Question"                                     # Note: "New Question" should match your button
    fill_in "Title", with: "Some valid question title"          # Note: "Title" should match the label for new questions
    fill_in "Description", with: "Some valid description"       # Note: "Description" should match the label for new questions
    click_on "Create Question"                                  # Note: "Create Question" should match your button
    expect(current_path).to eq(questions_path)
    expect(page).to have_text /Some valid question title/i
    # save_and_open_page
  end

  it "doesn't create a question with empty title" do
    visit new_question_path
    fill_in "Title", with: ""                                   # Note: "Title" should match your label
    fill_in "Description", with: "Some valid description"       # Note: Description should match your label
    click_on "Create Question"                                  # Note: "Create Question" should match your button
    expect(page).to have_text /Please correct the form/i
    expect(Question.count).to eq(0)
  end


end
```
Grab the [capybara-webkit](https://github.com/thoughtbot/capybara-webkit) gem and don't forget to add some configuration to the spec helper.  
```ruby
# spec/spec_helper.rb

# ...
  Capybara.javascript_driver = :webkit
  
  
  config.before(:suite) do
    DatabaseCleaner.strategy = :truncation
  end

  config.before(:each) do
    DatabaseCleaner.start
  end

  config.after(:each) do
    DatabaseCleaner.clean
  end
  
# ...

```  
And let's create a spec to check creating answers  
```ruby
# spec/features/creating_answers_spec.rb

require 'spec_helper'

feature "Creating an answer for a question" do

  let(:user) { create(:user) }
  let(:question) { create(:question) }

  before do
    login_as(user, scope: :user)
  end

  it "creates an answer for the qustion", js: true do
    visit question_path(question)
    fill_in "answer_body", with: "Some valid answer"
    click_on "Submit an answer"
    expect(page).to have_text /Some valid answer/
  end
end
```
If you are having issues with this, try setting this line in your spec helper from true to false.  
```ruby
# spec/spec_helper.rb

# ...

  config.infer_base_class_for_anonymous_controllers = false

# ...

```