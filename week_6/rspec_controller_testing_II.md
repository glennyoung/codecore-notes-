# RSpec: Controller Testing II  
  
Let's create a spec for our questions controller. First, comment out the questions controller  
```ruby
# app/controllers/questions_controller.rb

class QuestionsController < ApplicationController
#  before_action :authenticate_user!, except: [:index, :show]
#  before_action :find_question, 
#                  only: [:edit, :destroy, :update, :vote_up, :vote_down]
#
#  def index
#    @questions = Question.all
#  end
#
#  def new
#    @question = Question.new
#  end
#
#  def create
#    #@question = Question.new(params[:question]) # We used to be able to do this, but there were some security issues.
#    # now, in Rails 4, the default action is to prevent everything, rather than allowing. 
#    question_attributes = params.require(:question).permit([:title, :description])
#    @question = current_user.questions.new(question_attributes)
#
#
#    if @question.save
#      redirect_to questions_path, notice: "Your question was created successfully."
#    else
#      flash.now[:alert] = "Please correct the form"
#      render :new
#    end
#  end
#
#  def show
#    @question = Question.find(params[:id])
#    @answer   = Answer.new
#    @favorite = current_user.favorite_for(@question)
#    @vote     = current_user.vote_for(@question) || Vote.new
#    @answers  = @question.answers.ordered_by_creation
#  end
#
#  def edit
#  end
#
#  def update
#    if @question.update_attributes(question_attributes)
#      redirect_to @question, notice: "Question updated successfully"
#    else
#      flash.now[:alert] = "Couldn't update!"
#      render :edit
#    end
#  end
#
#  def destroy
#    if @question.destroy
#      redirect_to questions_path, notice: "Question deleted successfully."
#    else
#      redirect_to question_path, alert: "We had trouble deleting."
#    end
#  end
#
#  def vote_up
#    @question.increment!(:vote_count)
#    session[:has_voted] = true
#    redirect_to @question
#  end
#
#  def vote_down
#  end
#
#  def search
#  end
#
#  private
#
#  def question_attributes
#    params.require(:question).permit([:title, :description, {category_ids: []}])
#  end
#
#  def find_question
#    #@question = Question.find(params[:question_id] || params[:id])
#    # @question = current_user.questions.find(params[:question_id] || params[:id])
#    @question = current_user.questions.find_by_id(params[:id])
#    redirect_to root_path, alert: "Access Denied" unless @question
#  end


end
```
Now, we can define a test that is expecting index to exist.  
```ruby
# spec/controller/questions_controller_spec.rb
require 'spec_helper'

describe QuestionsController do

  describe "#index" do

    it "assigns a vaiable @questions" do
      get :index
      expect(assigns(:questions)).to be
    end

  end

end
```
We then can write the minimal amount of code needed to make the test pass.  
```ruby
# app/controllers/questions_controller.rb

class QuestionsController < ApplicationController
  
  def index
  end
  
end
```
Let's add a test case to check that an instance variable questions contains all the questions.
```ruby
# spec/controller/questions_controller_spec.rb

  # ...
  
    it "includes all questions in the database" do
      question = create(:question)
      get :index
      expect(assigns(:questions)).to include(question)
    end
  
  # ...
```
Now, to make this pass, we just add to our questions controller index method. 
```ruby
# app/controllers/questions_controller.rb

class QuestionsController < ApplicationController
  
  def index
    @questions = Question.all
  end
  
end
```
Let's test that our index template is being rendered.
```ruby
# spec/controller/questions_controller_spec.rb

  # ...
  
    it "renders index template" do
      get :index
      expect(response).to render_template(:index)
    end
    
  # ...
  
```
Let's try to test that we only return 10 questions at a time.  
```ruby

  # ...
  
# spec/controller/questions_controller_spec.rb
    it "only fetches 10 questions" do
      20.times { create(:questions) }
      get :index
      expect(assigns(:questions).length).to eq(10)
    end
    
  # ...

```
This will give an error, because FactoryGirl will be trying to create the same question 20 times. To avoid this, we can add a variable sequence to the questions factory
```ruby
# spec/factories/questions.rb

FactoryGirl.define do
  factory :question do
    sequence(:title) {|n| "#{Faker::Company.bs} #{n}"}
    description Faker::Lorem.sentence
  end
end

```
Let's add some more controller tests for the method "new"  
```ruby
# spec/controller/questions_controller_spec.rb

# ...

  describe "#new" do

    it "assigns a new question" do
      get :new
      expect(assigns(:question)).to be_a_new(Question)
    end

    it "renders :new template" do
      get :new
      expect(response).to render_template(:new)
    end
  end
  
# ...
```
Now, let's add contexts for signed in and not signed in, and another test. And also make a let to create a new user.    
```ruby
# spec/controller/questions_controller_spec.rb
describe QuestionsController do
  let(:user) { create(:user) }
  
# ...

  describe "#new" do

    context "with signed in user" do
      before { sign_in user }

      it "assigns a new question" do
        get :new
        expect(assigns(:question)).to be_a_new(Question)
      end

      it "renders :new template" do
        get :new
        expect(response).to render_template(:new)
      end
    end

    context "with signed out user" do
      it "redirects to sign in page" do
        get :new
        expect(response).to redirect_to(new_user_session_path)
      end
    end

  end
  
# ...

end
```
Let's add some tests to check the create method in our questions controller
```ruby

  describe "#create" do

    context "with no signed in user" do
      it "redirects to sign in page" do
        post :create
        expect(response).to redirect_to(new_user_session_path)
      end
    end

  end
```
To make this pass, just add the create method to your questions controller which should now be like this
```ruby
# app/controllers/questions_controller.rb

class QuestionsController < ApplicationController
  before_action :authenticate_user!, except: [:index]
  def index
    @questions = Question.limit(10)
  end

  def new
    @question = Question.new
  end

  def create

  end
end
```
Let's add some tests to our create method and another context for valid requests  
```ruby
  describe "#create" do

    context "with no signed in user" do
      it "redirects to sign in page" do
        post :create
        expect(response).to redirect_to(new_user_session_path)
      end

      context "with valid request" do
       it "creates a question in the databse" do
         expect do
           post :create, question: {title: "Valid Title", description: "valid question description"}
         end.to change { Question.count }.by(1)
       end
      end
    end
```  
To make this pass, just add to the create method in the questions controller.  
```ruby
# app/controllers/questions_controller.rb

# ...

  def create
    @questions = Question.create(params.require(:question).permit([:title, :description]))
    render nothing: true
  end
  
# ...
  
```
The whole spec file for your questions controller should look like this.
```ruby
# spec/controllers/questions_controller_spec.rb

require 'spec_helper'

describe QuestionsController do
  let(:user) { create(:user) }

  describe "#index" do

    it "assigns a variable @questions" do
      get :index
      expect(assigns(:questions)).to be
    end

    it "includes all questions in the database" do
      question = create(:question)
      get :index
      expect(assigns(:questions)).to include(question)
    end

    it "renders index template" do
      get :index
      expect(response).to render_template(:index)
    end

    it "only fetches 10 questions" do
      20.times { create(:question) }
      get :index
      expect(assigns(:questions).length).to eq(10)
    end

  end

  describe "#new" do

    context "with signed in user" do
      before { sign_in user }

      it "assigns a new question" do
        get :new
        expect(assigns(:question)).to be_a_new(Question)
      end

      it "renders :new template" do
        get :new
        expect(response).to render_template(:new)
      end
    end

    context "with signed out user" do
      it "redirects to sign in page" do
        get :new
        expect(response).to redirect_to(new_user_session_path)
      end
    end

  end

  describe "#create" do

    context "with no signed in user" do
      it "redirects to sign in page" do
        post :create
        expect(response).to redirect_to(new_user_session_path)
      end
    end

    context "with signed in user" do
      before { sign_in user }

      def valid_request
        post :create, question: {title: "Valid Title", description: "Valid Description"}
      end

      context "with valid request" do
       it "creates a question in the databse" do
         valid_request
         expect(response).to redirect_to(questions_path)
       end
      end
    end


  end


end
```
To make the last test pass just your questions controller should now look like this.  
```ruby
# app/controllers/questions_controller.rb

class QuestionsController < ApplicationController
  before_action :authenticate_user!, except: [:index]

  def index
    @questions = Question.limit(10)
  end

  def new
    @question = Question.new
  end

  def create
    @questions = Question.create(params.require(:question).permit([:title, :description]))
    redirect_to questions_path
  end
```
Add a spec to test for flash messages upon successful question creation.  
```ruby
# spec/controllers/questions_controller_spec.rb

# ...

       it "sets the flash message" do
         valid_request
         expect(flash[:notice]).to be
       end

# ...       
```
To make this pass, add the notice to the create method in the questions controller
```ruby
# app/controllers/questions_controller.rb

# ...

  def create
    @questions = Question.create(params.require(:question).permit([:title, :description]))
    redirect_to questions_path, notice: "question created successfully"
  end
  
# ...

```
Let's create a context for invalid request and make sure they do not change the question count.
```ruby
# spec/controllers/questions_controller_spec.rb

# ...

      context "with invalid request" do
        def invalid_request
          post :create, question: {title: "", description: "Valid Description"}
        end

        it "doesn't change the number of questions in the database" do
          invalid_request
          expect { invalid_request }.to_not change{Question.count}
        end
      end
      
# ...
```
This test should already pass. Let's add another test to the invalid context and one to the valid context.  
```ruby
# spec/controllers/questions_controller_spec.rb

# ...
  
  context "with valid request" do
  
  # ...
  
    it "assigns the questions to the signed in user" do
      expect { valid_request }.to change { user.questions.count }.by(1)
    end
    
  # ...
  
  end

# ...

  context "with invalid request" do
    
    # ...
    
    it "renders new template" do
      invalid_request
      expect(response).to render_template(:new)
    end
    
    # ...
    
  end
  
# ...

```
To make these pass, we can modify the create method in the questions controller appropriately.  
```ruby
# app/controllers/questions_controller.rb

# ...

  def create
    @question = Question.new(params.require(:question).permit([:title, :description]))
    @question.user = current_user
    if @question.save
      redirect_to questions_path, notice: "question created successfully"
    else
      render :new
    end
  end

# ...

```  
So far we've done the index, new, and create. Let's do show.
```ruby
# spec/controllers/questions_controller_spec.rb

# ...

  describe "#show" do
    let(:question) { create(:question) }

    context "with no signed in user" do

      it "assigns the question with the passed id" do
        get :show, id: question.id
        expect(assigns(:question)).to eq(question)
      end

    end
  end
  
# ...
  
```
To make this test pass, let's define a show method in the questions controller.  
```ruby
# app/controllers/questions_controller.rb

# ...

  def show
    @question = Question.find(params[:id])
  end

# ...
```

## Should look like this now
```ruby
# spec/controllers/questions_controller_spec.rb
require 'spec_helper'

describe CommentsController do
  let(:question) { create(:question) }
  let(:answer) { create(:answer) }
  let(:user) { create(:user) }

  describe "#create" do

    def valid_request
      post :create, answer_id: answer.id,
        comment: {body: "Valid body"}
    end

    def invalid_request
      post :create, answer_id: answer.id,
        comment: {body: ""}
    end


    context "without a signed in user" do
      it "doesn't create a comment" do
        expect { valid_request }.to_not change { Comment.count }
      end

      it "redirects unsigned in user to sign up page" do
        valid_request
        expect(response).to redirect_to(new_user_session_path)
      end
    end

    context "with disgned in user" do
      before { sign_in user }

      context "with valid request" do
        it "creates a comment in the database with valid params" do
          expect do
            expect { valid_request }.to change {Comment.count }.by(1)
          end
        end

        it "associates the comment with the answer whose id is passed" do
          expect do
            valid_request
          end.to change {answer.comments.count }.by(1)
        end
      end

      context "with invalid request" do
        it "redirects to the question page with invalid request" do
          expect { invalid_request }.to_not change{ Comment.count }
        end

        it "renders the question show page" do
          invalid_request
          expect(response).to render_template("questions/show")
        end

        it "sets flash alert" do
          invalid_request
          expect(flash[:alert]).to be
        end
      end

    end

  end


  describe "#destroy" do
    let!(:comment) { create(:comment, answer: answer)}

    def delete_request
      delete :destroy, answer_id: answer.id, id: comment.id
    end

    context "Without a signed in user" do

      it "redirects user to sign in" do
        delete_request
        response.should redirect_to(new_user_session_path)
        # expect(response).to redirect_to new_user_session_path
      end

      it "sets flash alert" do
        delete_request
        expect(flash[:alert]).to be
      end
    end

    context "with a signed in user" do
      before { sign_in user }
      it "should decrease number of comments by one" do
        expect { delete_request }.to change{Comment.count}.by(-1)
      end

      it "should redirect to the question" do
        # expect { delete_request }.to redirect_to question
        delete_request
        response.should redirect_to answer.question
      end

      it "sets a notice flash message" do
        delete_request
        expect(flash[:notice]).to be
      end
    end
  end

end
```
And the controller
```ruby
# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController
  before_action :authenticate_user!, except: [:index, :show]
  before_action :find_question,
                  only: [ :edit, :destroy, :update, :vote_up, :vote_down]

  def index
    @questions = Question.limit(10)
  end

  def new
    @question = Question.new
  end

  def create
    #@question = Question.new(params[:question]) # We used to be able to do this, but there were some security issues.
    # now, in Rails 4, the default action is to prevent everything, rather than allowing.
    question_attributes = params.require(:question).permit([:title, :description])
    @question = current_user.questions.new(question_attributes)
    if @question.save
      redirect_to questions_path, notice: "Your question was created successfully."
    else
      flash.now[:alert] = "Please correct the form"
      render :new
    end
  end

  def show
    @question = Question.find(params[:id])
    @answer = Answer.new
    if user_signed_in?
      @favorite = current_user.favorite_for(@question)
      @vote     = current_user.vote_for(@question) || Vote.new
    end
    @answers = @question.answers.ordered_by_creation
  end

  def edit
    @question = current_user.questions.find(params[:id])
  end

  def update
    if @question.update_attributes(question_attributes)
      redirect_to @question, notice: "Question updated successfully"
    else
      flash.now[:alert] = "Couldn't update!"
      render :edit
    end
  end

  def destroy
    if @question.destroy
      redirect_to questions_path, notice: "Question deleted successfully."
    else
      redirect_to question_path, alert: "We had trouble deleting."
    end
  end

  def vote_up
    @question.increment!(:vote_count)
    session[:has_voted] = true
    redirect_to @question
  end

  def vote_down
  end

  def search
  end

  private

  def question_attributes
    params.require(:question).permit([:title, :description, {category_ids: []}])
  end

  def find_question
    #@question = Question.find(params[:question_id] || params[:id])
    # @question = current_user.questions.find(params[:question_id] || params[:id])
    @question = current_user.questions.find(params[:id])
    redirect_to root_path, alert: "Access Denied" unless @question
  end

end
```
The user factory should look like this.  
```ruby
FactoryGirl.define do
  factory :user do
    first_name Faker::Name.first_name
    last_name  Faker::Name.last_name
    sequence(:email) {|n| "some_email#{n}@awesomeanswers.com" }
    password Faker::Internet.password
  end
end

```
And the question factory should look like this.  
```ruby
FactoryGirl.define do
  factory :question do
    association :user, factory: :user
    sequence(:title) {|n| "#{Faker::Company.bs} #{n}"}
    description Faker::Lorem.sentence
  end
end
```  
## After Lunch: Update, etc.  
Let's add a spec to test an update action in the questions controller.  
```ruby
# spec/controllers/questions_controller_spec.rb

# ...

  describe "#update" do
    context "with signed in user" do
      before { sign_in user }

      it "updates the question with new title" do
        patch :update, id: question.id, question: {title: "Some new title"}
        question.reload
        expect(question.title).to match /some new title/i
      end

      it "redirects to question show page after successful update" do
        patch :update, id: question.id, question: {title: "Some new title"}
        expect(response).to redirect_to(question)
      end

      it "sets flash message with successful update" do
        patch :update, id: question.id, question: {title: "some title"}
        expect(flash[:notice]).to be
      end

      it "renders edit template with failed update" do
        patch :update, id: question.id, question: {title: ""}
        expect(response).to render_template(:edit)
      end

      it "raises an error if trying to update another user" do
          expect do
            patch :update, id: question.id1, question: {title: "some new title"}
          end.to raise_error
      end

    end
  end

# ...

```
and the questions controller update action  
```ruby
# app/controllers/questions_controller.rb

# ...

  def update
    if @question.update_attributes(question_attributes)
      redirect_to @question, notice: "Question updated successfully"
    else
      render :edit
    end
  end
  
  private

  def question_attributes
    params.require(:question).permit([:title, :description, {category_ids: []}])
  end
  
# ...
```
Let's make a spec to test a delete action
```ruby
# spec/controllers/questions_controller_spec.rb

# ...

  describe "#destroy" do

    context "with signed in user" do
      before { sign_in user}
      let!(:question) { create(:question, user: user) }

      it "removes the qustion from the database" do
        expect do
          delete :destroy, id: question.id
        end.to change { Question.count }.by(-1)
      end

      it "redirect to questions listing page with success" do
        delete :destroy, id: question.id
        expect(response).to redirect_to(questions_path)
      end

      it "it raises error when trying to delete questions by non-owners" do
        expect do
          delete :destroy, id: question1.id
        end.to raise_error
      end
    end
  end

# ...

```
And the questions controller destroy method
```ruby
# app/controllers/questions_controller.rb
  before_action :find_question,
                  only: [ :edit, :destroy, :update, :vote_up, :vote_down]
# ...

  def destroy
    if @question.destroy
      redirect_to questions_path, notice: "Question deleted successfully."
    else
      redirect_to question_path, alert: "We had trouble deleting."
    end
  end
  
# ...

  private
  
  def find_question
    @question = current_user.questions.find(params[:id])
    redirect_to root_path, alert: "Access Denied" unless @question
  end
  
# ...
```