# Polymorphic Association  
[Polymorphic Association](http://en.wikipedia.org/wiki/Polymorphic_association) | [Single Table Inheritence](http://en.wikipedia.org/wiki/Single_Table_Inheritance) | [Extra Practice](http://robots.thoughtbot.com/using-polymorphism-to-make-a-better-activity-feed-in-rails) | [Read this Page!](http://guides.rubyonrails.org/association_basics.html) | [Old Railscast](http://railscasts.com/episodes/154-polymorphic-association)  
![taging polymorphic association](http://i.stack.imgur.com/KdqIt.jpg)  
## Single Table Inheritance  
  
"Single table inheritance is a way to emulate object-oriented inheritance in a relational database. Polymorphic association is a term used in discussions of Object-Relational Mapping with respect to the problem of representing in the relational database domain, a relationship from one class to multiple classes." (ref: wikipedia)  
  
"Relational databases don't support inheritance, so when mapping from objects to databases we have to consider how to represent our nice inheritance struc-tures in relational tables. When mapping to a relational database, we try to minimize the joins that can quickly mount up when processing an inheritance structure in multiple tables. Single Table Inheritance maps all fields of all classes of an inheritance structure into a single table." - [Martin Fowler](http://www.martinfowler.com/eaaCatalog/singleTableInheritance.html)  
  
Create a new rails app and generate a user model.
```bash
rails new test_sti -T
model user first_name last_name email type
cd test_sti
rake db:migrate
```
Create a new model for Moderator and User, both of which inherit from User. However, we'll create them in a separate folder, in models folder. Rails will treat this as a module, and we'll be able to access these models double colon style: `User::Moderator`.  
  
**Note**: because we didn't do this first, we actually moved the models into a new folder, we need to update our moderators: `User.where(type: "Moderator").update_all(type: "User::Moderator")`  
  
```ruby
# app/models/user/moderator.rb

class Moderator < User
end
```
```ruby
# app/models/user/admin.rb

class Admin < User
end
```
Now, we can add some validations to the different user models
```ruby
# app/models/user.rb

class User < ActiveRecord::Base
  validates_presence_of :email
end
```
```ruby
# app/models/user/moderator.rb

class Moderator < User
  validates_presence_of :first_name, :last_name

end
```
```ruby
# app/models/user/admin.rb

class Admin < User

end
```
## Polymorphic Associations
Let's generate some resources and make them 'commentable'
```
rails generate resource discussion title body:text user:references
rails generate resource comment body:text commentable:references
```
Open up the comments migration file and change `index: true` to `polymorphic: true`, and add indexes for commentable id and type.
```ruby
class CreateComments < ActiveRecord::Migration
  def change
    create_table :comments do |t|
      t.text :body
      t.references :commentable, polymorphic: true

      t.timestamps
    end
    add_index :comments, [:commentable_id, :commentable_type]
  end
end
```
Now we can do `rake db:migrate`  
  
  
Add some commentable and comment associations to the comment, user, campaign and discussion models.
```ruby
# app/models/comment.rb

class Comment < ActiveRecord::Base
  belongs_to :commentable, polymorphic: true
end
```
```ruby
# app/models/user.rb

has_many :comments, as: :commentable
```
```ruby
# app/models/discussion.rb

has_many :comments, as: :commentable
```

Now, we can add comments to discussions in rails console.  
```
d = Discussion.create(title: "jkahsdkjsadh", body: "kjahsdkjashdkajshdsakjhsad")
d.comments.create(body: "akfjkl asdfl;j sadflkjsdaf ")
```
Add to the routes.  
```ruby

# ...


  resources :discussions do
    resources :comments
  end
  
# ...

```
Add a create method to the comments controller.  
```ruby
# app/controllers/comments_controller.rb

class CommentsController < ApplicationController
  before_action :authenticate_user!

  def create
    @campaign = Campaign.find(params[:campaign_id])
    @comment = @campaign.comments.new(comment_params)
    if @comment.save
      redirect_to @campaign, notice: "Comment created"
    else
      render "/campaigns/show"
    end
  end

  private

  def comment_params
    params.require(:comment).permit(:body)
  end
end
```
Add an HTML view to show the comment
```haml
/ app/views/campaigns/show.html.haml

- c = @campaign
%h1= c.title
%p= c.details
%div{class: label_class(@campaign.state)}
  = c.state.capitalize
%hr

= simple_form_form [@campaign, @comment] do |f|
  = f.input :body
  = f.submit "submit", class: "btn btn-default"

- c.comments.each do |comment|
  .well= comment.body
```
```
# app/models/campaign.rb

class Campaign < ActiveRecord::Base
  belongs_to :user
  has_many :comments, as: :commentable

  scope :published, -> { where(state: :published) }

  state_machine :state, initial: :draft do

    event :publish do
      transition draft: :published
    end

    event :complete do
      transition published: :target_met
    end

    event :expire do
      transition target_met: :succeeded, published: :failed
    end

    event :cancel do
      transition [:draft, :published, :target_met] => :canceled
    end

    after_transition on: :canceled, do: :refund_all_pledges
  end

  def refund_all_pledges
    # code to schedule bg task to refund all
  end
end
```
Now we can add a commentable instance variable to our comments controller
```ruby
# app/controllers/comments_controller.rb

class CommentsController < ApplicationController
  before_action :find_commentable
  before_action :authenticate_user!

  def create
    @campaign = Campaign.find(params[:campaign_id])
    @comment = @campaign.comments.new(comment_params)
    if @comment.save
      redirect_to @campaign, notice: "Comment created"
    else
      render "/#{@commentable.class.name.underscore.pluralize}/show"
    end
  end

  private

  def comment_params
    params.require(:comment).permit(:body)
  end

  def find_commentable
    resource, id = request.path.split("/")[1, 2]
    @commentable = resource.singularize.classify.constantize.find(:id)
    if params[:campaign_id]
      @commentable = Campaign.find(params[:campaign_id])
    elsif params[:discussion_id]
      @commentable = Discussion.find(params[:discussion_id])
    end
  end
end
```

Add validation for body to the comment model
```ruby
# app/models/comment.rb

class Comment < ActiveRecord::Base
  belongs_to :commentable, polymorphic: true
  validates_presence_of :body
end
```
Add commentable to the show action in the campaigns controller.  
```ruby
# app/controllers/campaigns_controller.rb

# ...

  def show
    @campaign = Campaign.find(params[:id])
    @commentable = @campaign
    @comment = Comment.new
    respond_to do |format|
      format.html { render :show }
      format.js { render :show }
    end
  end
  
# ...
```
Now, have a look at scaffold!
```
rails generate scaffold_controller discussions title body:test
```
Make a form partial in your comments views
```haml
/ app/views/comments/_listing_with_new.html.haml

= simple_form_for [@commentable, @comment] do |f|
  = f.input :body
  = f.submit "submit", class: "btn btn-default"

- @commentable.comments.each do |comment|
  .well= comment.body
```
In the discussion show view render this partial
```haml
/ app/views/discussons/show.html.haml

%p#notice= notice

%p
  %b Title:
  = @discussion.title
%p
  %b Body:
  = @discussion.body

= link_to 'Edit', edit_discussion_path(@discussion)
\|
= link_to 'Back', discussions_path

%hr

= render "/comments/listing_with_new"
```
And in the campaigns show view as well
```haml
/ app/views/campaigns/show.html.haml

- c = @campaign
%h1= c.title
%p= c.details
%div{class: label_class(@campaign.state)}
  = c.state.capitalize
%hr

= render "/comments/listing_with_new"
```