**

(project)1—n(members)n—1(users)

Project has many users through members
Users has many projects through members

**rails g model members user:references project:references**

**bin/rake db:migrate**


**project.rb**

```ruby
has_many :members, dependent: :destroy
has_many :projectmembers, through: :members, source: :user
```

**user.rb**
```ruby
has_many :members, dependent: :destroy
has_many :projectwm, through: :members, source: :project
```

We don’t need a controller on this case since all we do is create/edit on the project create page.


** form_page **

```ruby
<%= f.label :projectmembers_ids %>
<%= f.collection_check_boxes :projectmember_id, User.all, :id, :full_name %>
```

User.all: list of all objects that u want to display
:id: the value we want to send to the server
:full_name: display function


** user.rb **
```ruby
def full_name
     if first_name || last_name
     “#{first_name} #{last_name}”.squeeze(" ").strip
     else
          email
     end
end
```

**Questions controller**
`
add {projectmember_ids: []} to strong params or else it will not save
`

:: MANY TO MANY w/model referencing itself

A user follows another user

(user)1——n(following)n——1(user)

** rails g model following user:references follower_id:integer **

** rake db:migrate **

** following.rb **
```ruby
belongs_to :user
belongs_to :follower, class_name: “User”   
```
“User” class is to make an explicit reference to User, since the relationship from following to user is referred to twice.

** user.rb **
```ruby
has_many :followings, dependent: :destroy
has_many :followers, through :followings

has_many :inverse_followings, class_name: “Following”, foreign_key: “follower_id"
has_many :inverse_followers, through: :inverse_followings, source: :user
```
We specify the foreign_key follower_id because we don’t want to refer user_id twice.

u = User.last
u. followers
-> INNER JOIN

u.inverse_followers
-> INNER JOIN but instead of follower_id it uses user_id
