# Google Maps for Rails
[Google Maps](https://github.com/apneadiving/Google-Maps-for-Rails) |  
Install the gmaps4rails gem
```ruby
# Gemfile

gem 'gmaps4rails'
gem 'underscore-rails'
gem "geocoder"
```
Create a migration file and a controller
```
rails generate migration AddLatitudeAndLongitudeToCampaign latitude:float longitude:float full_street_address:string
rails generate controller nearby_campaigns index
```
Add some javascripts to your application layout
```erb
<script src="//maps.google.com/maps/api/js?v=3.13&amp;sensor=false&amp;libraries=geometry" type="text/javascript"></script>
<script src='//google-maps-utility-library-v3.googlecode.com/svn/tags/markerclustererplus/2.0.14/src/markerclusterer_packed.js' type='text/javascript'></script>
```
Add to your assets pipeline
```javascript
// app/assets/javascripts/application.js

//= require underscore
//= require gmaps/google

```
geocoded_by :full_street_address

```haml
= link_to "Nearby Campaigns", 
```
```erb
<div style='width: 800px;'>
  <div id="map" style='width: 800px; height: 400px;'></div>
</div>
```
or
```haml
%div{style: "width: 800px; height: 420px;"}
  #map{style: "width: 800px; height: 400px"}
```
```ruby
# app/models/campaign.rb

# ...

geocoded_by :full_street_address   # can also be an IP address
after_validation :geocode

# ...
```
To the user model add delegate address
```ruby
  delegate :full_name, :address, to: :profile
```
Add a method to your decorator
```ruby
# app/decorators/campaign_decorator.rb

# ...

  def gen_marker(marker)
    marker.lat object.latitude
    maker.lng object.longitude
    link = h.link_to object.title, object
    info = "#{link}<br>#{short_details}"
    marker.infowindow info
  end

# ...
```
```haml
/ app/views/nearby_campaigns/index.html.haml

%div{style: "width: 800px"}
  #map{style: "width: 800px; height: 400px"}

 - markers = @hash = Gmaps4rails.build_markers(@users) { |campaign, marker| campaign.decorate.gen_marker(marker) }


:javascript
  $(document).ready(function() {
    handler = Gmaps.build('Google');
    handler.buildMap({ provider: {}, internal: {id: 'map'}}, function(){
      markers = handler.addMarkers(#{markers.to_json});
    handler.bounds.extendWith(markers);
    handler.fitMapToBounds();
    });
  });
```

```ruby
# app/controllers/nearby_campaigns_controller.rb

class NearbyCampaignsController < ApplicationController
  def index
    @nearby_campaigns = Campaign.near(current_user.address, 30)
  end
end
```

```ruby
# app/helpers/application_helper.rb

# ...

  def generate_gmap4rails_markers(objects)
    Gmaps4rails.build_markers(@users) do |object, maker|
      object.decorate.gen_marker(marker)
    end
  end

# ...
```

## Services
[Virtus](https://github.com/solnic/virtus)
  
Create a services folder in the app directory. We'll refactor our campaign create action to use a service object. 
```ruby
# app/controllers/ccampaigns_controller.rb

# ...

  def create
    service = Campaign::CreateCampaign.new(params: campaign_params, user: current_user)
    # @campaign = Campaign.new(campaign_params)
    # @campaign.user_id = current_user.id
    if service.call
      redirect_to service.campaign, notice: "success"
    else
      @campaign = service.campaign
      (3 - @campaign.reward_levels.length).times { @campaign.reward_levels.build }
      render :new
    end
  end

# ...

``` 
```ruby
# app/services/campaign/create_campaign.rb

class Campaign::CreateCampaign

  include Virtus.model

  attribute :params, Hash
  attribute :user, User
  attribute :campaign, Campaign

  def call
    @campaign = user.campaigns.new(params)
    @campaign.save
  end

end
```

In the application settings create this line
```ruby
# config/application.rb

# ...

config.autoload_paths += Dir[Rails.root.join('app', 'services','*').to_s]

# ...
```