# Mongo DB  
[mongoid docs](http://mongoid.org/en/mongoid/index.html) |  [embeded](http://mongoid.org/en/mongoid/docs/relations.html#embeds_one) | [referenced](http://mongoid.org/en/mongoid/docs/relations.html#has_one) | [MongoDB Data Modeling](http://docs.mongodb.org/manual/data-modeling/) | [Admin UI](http://docs.mongodb.org/ecosystem/tools/administration-interfaces/)  
  
Mongo DB is schemaless. This means we actually have no table. In a SQL database, we have to create tables to insert information. With Mongo, there are no tables. It's document based.  
  
This basically means that instead of inserting a row, we're actually creating a document. This means that without having to put tables, there's no migrations. This makes things easier from an ActiveRecord point of view. You can pretty much put anything in the document.  
  
If you've already entered data in some column, and you remove it, you won't have access to it with attr_accessor, becuase there's no column in the database.  
  
The format it stores the data on is called BSON (Binary JSON). BSON is basically like JSON, but we put it in binary form. Think of this like those JSON files we dealt with. The best way to think of it is like a ruby hash, or associative array in JavaScript.  
  
```ruby
{ 
  name: "Tam",
  age: 15
}
```  
[Install Mongo DB](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-os-x/) through brew `brew install mongodb`.  
  
Make a directory for the database and [change ownership](http://ss64.com/bash/chown.html) on it to your username
```bash
sudo mkdir -p /data/db
sudo chown [your-user-name] /data/db        # check your username with whoami
```  
You can then start a Mongo DB server with `mongod`.  
  
Now, make a new rails application `rails new mongo_app --skip-active-record` and specify to skip the active record.  
Add a gem `gem mongoid` to your Gemfile  
```
gem 'mongoid', git: 'https://github.com/mongoid/mongoid'  
```  
  
Then bundle  and [config mongoid](http://mongoid.org/en/mongoid/docs/installation.html#installation) `rails generate mongoid:config`  
  
Now you can generate a model: `rails g resource post title description`  
  
If you want to push this to Heroku, you will have to go to [addons for mongodb](https://devcenter.heroku.com/articles/mongohq), because the default is Postgres.  
  
```ruby
post {
  _id : 01,
  title: 'Hey there',
  description: 'this is it',
  comments: [ 
    { body: 'kjasdkjasdkj '},
    { body: 'kj adslkj adskj'}
  ],
  date: '03-05-2007'
}
```  
Generate a controller: `rails g controller posts` and open up the file  
```ruby
# app/contollers/posts_controller.rb
class PostsController < ApplicationController

  def index
    @posts = Post.all
  end

  def new
    @post = Post.new
  end

  def create
    @post = Post.new(post_params)
    if @post.save
      redirect_to @post, notice: "Post created successfully."
    else
      render :new
    end
  end

  def show
    @post = Post.find(params[:id])
    @comment = Comment.new
  end

  private

  def post_params
    Post.new(params.require(:post).permit([:title, :description]))
  end

end
```  
add `gem 'haml-rails` to your Gemfile. and create a form for new posts.  
```haml
/ app/views/posts/new.haml.html
= form_for @post do |f|
  = f.label :title
  = f.text_field :title
  %br
  = f.label :body
  = f.text_area :body
  %br
  = f.submit "Save"
```  
Add `validates_presence_of :title` to your post model.  
```ruby
# app/models/post.rb
class Post
  include Mongoid::Document
  field :title, type: String
  field :description, type: String

  validates_presence_of :title

end
```  
Add a 'show' view.  
```haml
/ app/views/posts/show.html.haml
%h1= @post.title
%p= @post.description
```
  
Add comments `rails g resource comment body` with `embedded_in` just kidding, with `belongs_to`    
```ruby
# app/models/comment.rb

class Comment
  include Mongoid::Document
  field :body, type: String

  #embedded_in :post
  belongs_to :post
end

```  
Add `embeds :comment` to Post model, just kidding, add `has_many` instead  
```
# app/models/post.rb
class Post
  include Mongoid::Document
  field :title, type: String
  field :description, type: String

  validates_presence_of :title
  #embeds :comment
  has_many :comments
end
```
Then add it as a resource under posts in your routes.rb  
```ruby
# app/config/routes.rb

MongoApp::Application.routes.draw do

  resources :posts do
    resources :comments
  end
  
  root "posts#index"

end
```
add a create method to the comments controller  
```ruby
# app/controllers/comments_controller.rb
class CommentsController < ApplicationController

  def create
    @post = Post.find(params[:post_id])
    @comment = @post.comments.new(params.require(:comment).permit[:body])
    if @comment.save
      redirect_to @post, notice: "Comment created successfully"
    else
      render "/posts/show"
    end
end
```
  
Modify the post show view to include a form to add and wasy to view comments.  
```haml
%h1= @post.title
%p= @post.description
%hr
%h2 Comments
= form_for [@post, @comment] do |f|
  = f.text_area :body
  %br
  = f.submit "Comment"

- @post.comments.each do |comment|
  %p= comment.body
  %hr
```  