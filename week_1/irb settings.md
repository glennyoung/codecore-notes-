## Settings for IRB colors
 

To get some nice colors in your irb, you can install the gems `awesome_print` and `interactive_editor`. There should be a dot file (hidden file) in your home directory called .irbrc, if not just create it: `sublime ~/.irbrc`.  
```
gem install awesome_print
gem install interactive_editor
sublime ~/.irbrc
```  
[Tam's settings](https://gist.github.com/tkbeili/8423267)   
```ruby
# ruby 1.8.7 compatible
require 'rubygems'
require 'irb/completion'
 
# interactive editor: use vim from within irb
begin
  require 'interactive_editor'
rescue LoadError => err
  warn "Couldn't load interactive_editor: #{err}"
end
 
# awesome print
begin
  require 'awesome_print'
  AwesomePrint.irb!
rescue LoadError => err
  warn "Couldn't load awesome_print: #{err}"
end
 
# configure irb
#IRB.conf[:PROMPT_MODE] = :SIMPLE
 
# irb history
IRB.conf[:EVAL_HISTORY] = 1000
IRB.conf[:SAVE_HISTORY] = 1000
IRB.conf[:HISTORY_FILE] = File::expand_path("~/.irbhistory")
 
# load .railsrc in rails environments
railsrc_path = File.expand_path('~/.irbrc_rails')
if ( ENV['RAILS_ENV'] || defined? Rails ) && File.exist?( railsrc_path )
  begin
    load railsrc_path
  rescue Exception
    warn "Could not load: #{ railsrc_path } because of #{$!.message}"
  end
end
 
class Object
  def interesting_methods
    case self.class
    when Class
      self.public_methods.sort - Object.public_methods
    when Module
      self.public_methods.sort - Module.public_methods
    else
      self.public_methods.sort - Object.new.public_methods
    end
  end
end
 
module Kernel
  def require_relative(file)
    $:.unshift Dir.pwd
    require file
  end
 
  def guid(s)
    s.scan(/[a-f0-9-]{36}/).first
  end
end
```