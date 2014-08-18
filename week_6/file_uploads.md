# Uploading Files with Paperclip and Carrier Wave  
[Paperclip](https://github.com/thoughtbot/paperclip#image-processor)

```ruby
brew install imagemagick
brew install gs
```
Add paperclip to the Gemfile
```ruby
# Gemfile

# ...

gem "paperclip", "~> 4.1"

# ...
```  
Also, it's a good idea to add to the git ignore file so it does not include images.  
```gitignore
# .gitignore

# ...

/public/system

# ...
```
Create a migration to add an attachment: `rails generate migration add_attachment_to_questions`, then open the file and add
```ruby
class AddAttachmentToQuestions < ActiveRecord::Migration
  def change
    add_attachment :questions, :image
  end
end
```

Copy these lines and put it in your [question model](https://github.com/thoughtbot/paperclip#models)  
```ruby
# app/models/question.rb

# ...
  has_attached_file :image, styles: { :medium => "300x300>", :thumb => "100x100>" }, default_url: ActionController::Base.helpers.asset_path("missing_:style.png")
  validates_attachment_content_type :avatar, :content_type => /\Aimage\/.*\Z/

# ...
```
Run a `brake db:migrate`, then add an image upload field to your form
```haml
/ app/views/questions/_form.html.haml

= simple_form_for @question, html: {class: "form-horizontal"}  do |f|
  = f.input :title, label: "question title"
  = f.input :description, label: "questions description"
  = f.collection_check_boxes :category_ids, Category.order("title"), :id, :title, item_wrapper_class: "form-group"
  = f.file_field :image
  = f.submit class: "btn btn-default"
```
Add an image param to the permitted fields in your questions controller
```ruby
# app/controllers/questions_controller.rb

# ...

  def question_attributes
    params.require(:question).permit([:title, :description, {category_ids: []}, :image])
  end
  
# ...
  
```
Add an if statement to your questions show view to display an image if it's present.  
```haml
/ app/views/questions/show.html.haml

- if @question.image.present?
  = image_tag @question.image.url(:medium)
```
If you want to use something like Heroku, you cannot upload images directly to their servers. However, you can use something like [Amazon Web Services](http://aws.amazon.com/). They will ask you for a credit card when you sign up, but will only charge you if you use more than something like 5GB/month (disclaimer: idk what the contract with them is, please read any agreement and use at your own risk).  
  
```
# .gitignore
/config/initializers/s3_vars.rb
```
Then create that file we are going to ignore  
```ruby
ENV['s3_access_key_id'] = 'MYACCESSKEY'
ENV['s3_secret_key_id'] = 'MYSECRETKEYKASDKJHDSA'
```
Add the [aws-sdk](http://aws.amazon.com/sdkforruby/) gem `gem 'aws-sdk', '~> 1.0'` and run bundle install  

***aside***: check this out
```
hash1 = {a: 1, b:2}
hash2 = {c: 3, d: 4}
hash1.merge(hash2)
```
Back to the regularly scheduled show...   
  
Add a file to set up paperclip using s3
```ruby
# config/initializers/setup_paperclip.rb

Paperclip::Attachment.default_options.merge!({
  storage: :s3,
  s3_credentials: {
    access_key_id: ENV['s3_access_key_id'],
    secret_access_key: ENV['s3_secret_key_id'],
    bucket:        'the-name-of-your-bucket'
  }
});
```
Create a bucket on Amazon using s3...  
## CarrierWave
[carrierwave](https://github.com/carrierwaveuploader/carrierwave)  
Add some gems
```ruby
# Gemfile

# ...

gem 'carrierwave'
gem 'rmagick', require: 'RMagick'
gem 'fog'

# ...
```
run bundle install and generate an uploader  
```bash
bundle install
rails g uploader image
```
You can then set up everything in the folder it gives you.  
```ruby
# app/uploaders/image_uploader.rb

# encoding: utf-8

class ImageUploader < CarrierWave::Uploader::Base

  # Include RMagick or MiniMagick support:
  include CarrierWave::RMagick             # un comment out this line! 
  # include CarrierWave::MiniMagick

  # Choose what kind of storage to use for this uploader:
  storage :file
  # storage :fog

  # Override the directory where uploaded files will be stored.
  # This is a sensible default for uploaders that are meant to be mounted:
  def store_dir
    "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
  end

  # Provide a default URL as a default if there hasn't been a file uploaded:
  # def default_url
  #   # For Rails 3.1+ asset pipeline compatibility:
  #   # ActionController::Base.helpers.asset_path("fallback/" + [version_name, "default.png"].compact.join('_'))
  #
  #   "/images/fallback/" + [version_name, "default.png"].compact.join('_')
  # end

  # Process files as they are uploaded:
  # process :scale => [200, 300]
  #
  # def scale(width, height)
  #   # do something
  # end

  # Create different versions of your uploaded files:
   version :thumb do
     process :resize_to_fit => [50, 50]
   end

  # Add a white list of extensions which are allowed to be uploaded.
  # For images you might use something like this:
  # def extension_white_list
  #   %w(jpg jpeg gif png)
  # end

  # Override the filename of the uploaded files:
  # Avoid using model.id or version_name here, see uploader/store.rb for details.
  # def filename
  #   "something.jpg" if original_filename
  # end

end
```
Now, the only thing we really need to add in a migration to add images to questions
```
rails g migration add_image_to_questions image
rake db:migrate
```
In the question model comment out the paperclip stuff, and add something for CarrierWave
```ruby
# app/models/question.rb

# ...

#  has_attached_file :image, styles: { :medium => "300x300>", :thumb => "100x100>" }, default_url: ActionController::Base.helpers.asset_path("missing_:style.png")
#  validates_attachment_content_type :image, :content_type => /\Aimage\/.*\Z/

# ...

  mount_uplaoder :image, ImageUploader
```

Let's take a look at [Fog](http://fog.io/about/provider_documentation.html).  
```ruby
# app/uploaders/image_uploader.rb

class ImageUploader < CarrierWave::Uploader::Base

  include CarrierWave::RMagick

  storage (Rails.env.production? ? :fog : :file)

  def store_dir
    "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
  end


   version :thumb do
     process :resize_to_fit => [50, 50]
   end

end
```

```ruby
# config/initializers/setup_fog.rb

CarrierWave.configure do |config|
  config.fog_credentials = {

    provider: "AWS",
    aws_access_key_id:     ENV['s3_secret_key_id'],
    aws_secret_access_key: ENV['s3_secret_key_id']
  }

  config.fog_directory = ENV['s3_bucket']
  config.fog_public        = false
end
```

```ruby
# config/initializers/setup_paperclip.rb

if Rails.env.production? || Rails.env.staging?
  Paperclip::Attachment.default_options.merge!({
    storage: :s3,
    s3_credentials: {
      access_key_id: ENV['s3_access_key_id'],
      secret_access_key: ENV['s3_secret_key_id'],
      bucket:        "my-bucket-name"
    }
  });
end
```
