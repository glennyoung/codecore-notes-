# jQuery & CoffeeScript  
Open awesome answers... and use the js console in your browser
```javascript
$('img').animate({width: "50"});
$('img').animate({width: "100", height: "200"});
$('img').animate({width: "+=100"});
$('img').animate({width: "50%"});
$('img').animate({width: "-=10", height: "-=10"});
```  
Another thing we can check with animate, is how long it is animating for.  
```javascript
$('img').animate({width: "100", height: "200"}, 5000);
```  
And the last thing we can do is pass in a function to execute after the animation has happened.  
```javascript
$('img').animate({width: "100", height: "200"}, 5000, function(){alert("animation complete")});
$('img').animate({width: "100", height: "200"}, 5000, function(){alert("animation complete")});

// We can select by element, class, id
$('.btn').fadeOut(5000, function(){console.log("finished")});
$('.btn').slideUp();
$('.btn').slideDown();
$('.btn').slideDown(5000, function(){console.log("finished");});
```  
1.) Create a function that scales a picture up, then fades it out.  
```javascript
var bigFade = function() {
  $('img').animate({width: "500%", height: "500%", opacity: "0.25"}, 5000, function(){ $(this).fadeOut(2000);});
}

// alternative method
$('img').animate({width: "100%", height: "100%"}, 1000, function() {$(this.fadeOUt() });
```  
##jQuery Effect  
[jQuery Effect](http://jqueryui.com/effect/)  
  
Add the jquery ui rails gem to your gemfile.  
```ruby
# Gemfile

# ...

gem 'jquery-ui-rails'

# ...
```  
Then require jquery ui in your application.css and application.js files  
```css
* app/assets/stylesheets/application.css
* ...
*= require jquery.ui.all
* ...
```  
```javascript
// app/assets/javascripts/application.js
// ...
//= require jquery.ui.all
// ...
```  
2.) Write a function that increases the size of an image 100%, bounces it, then fades out.  
```javascript
var bigBounceFade = function() {
  $('img').animate({ width: "100%", height: "100%" }, 3000, function() { $(this).effect("bounce", 5000, function(){$(this).fadeOut(2000);});});
  };
```  
More effects  
```javascript
$('img').draggable({stop: function(){alert("drag complete");});
```  
3.) Write some jQuery that makes the content of the div id="text" to be "dragging" while you drag the image, and "done dragging" when you stop.   
```javascript
$('img').draggable({start: function(){
  $('body').find("#text").text("dragging")
  }, stop: function(){
    $('body').find("#text").text("done dragging");
    }
  });
```  
4.) You have a list. When you drag an item in the list, it fades away and disappears. Try to create this using jQuery.  
```javascript
$('li').draggable({stop: function(){$(this).fadeOut();}});
```  
5.) Have a page with many randomly generated buttons, and when you click on one, it generates a random [jQuery Effect](http://jqueryui.com/effect/)    
```javascript  
  var buttonGenerator = function(n) {
    for(var i = 0; i < n; i++) {
      $('body').append('<button class="effect btn btn-primary">Click Me</button>');
    }
  }
  buttonGenerator(Math.floor((Math.random()*100)+1));

  effects = [
    'blind',
    'bounce',
    'clip',
    'drop',
    'explode',
    'fade',
    'fold',
    'highlight',
    'puff',
    'pulsate',
    'scale',
    'shake',
    'size',
    'slide',
    'transfer'
    ]

  randomEffect = effects[Math.floor(Math.random() * effects.length)];

  $('.effect').on('click', function(){$(this).effect(randomEffect)});
```  
## CoffeeScript  

CoffeeScript has a ruby-like syntax. Like HAML, it is very case sensitive, and there are times when errors can be vague. It is case-sensitive, and whitespace sensitive. Try practicing using [js2coffee](http://js2coffee.org/).      
  
```javascript
// JavaScript
var a, b, sayHello;

a = void 0;

b = void 0;

sayHello = void 0;

a = 10;

if (a > b) {
  b = 5;
}

sayHello = function(name) {
  return alert("hello there, " + name);
};

$("hello").on("click", function() {
  return $(this).fadeOut(1000, function() {
    return $(this).effect("shake");
  });
});

// CoffeeScript
a = undefined
b = undefined
sayHello = undefined
a = 10
b = 5  if a > b

sayHello = (name) ->
  alert "hello there, " + name

$("hello").on "click", ->
  $(this).fadeOut 1000, ->
    $(this).effect "shake"

```  
***note***: Comments in CoffeeScript use a hashtag ('#').  
Doing Document Ready in CoffeeScript uses the '$'.  Instead of function, we use '->', and if we want to pass parameters, like function(myParam1, myParam2), we can like '(myParam1, myParam2) ->' and instead of 'this' we use '@'.
```javascript



```  
6.) Using coffee script, write some code so that when you click on a button, it toggles 'btn-primary' and 'btn-danger'.  
```coffeescript
$ ->
  $(".btn").on "click", ->
    $(@).toggleClass("btn-primary").toggleClass("btn-danger")

```  
Loops and arrays in CoffeeScript  
```coffeescript
# coffeescript
array = [1, 2, 3, 4, 5]

multiply = (x) -> x * x

multiply x for x in array

// javascript
var array, multiply, x, _i, _len;

array = [1, 2, 3, 4, 5];

multiply = function(x) {
  return x * x;
};

for (_i = 0, _len = array.length; _i < _len; _i++) {
  x = array[_i];
  multiply(x);
}
```  
7.) Create a method in coffeescript that capitalizes the first letter of each word in a string.  
```coffeescript
$ ->
  capitalize = (string) ->
  string.charAt(0).toUpperCase() + string.slice(1)

  $("#my-input").on "keyup", ->
    word_array = $(@).val().split(" ")
    word_array = word_array.map (word) -> capitalize(word)
    $("#shuffled").text word_array.join(" ")
```  
We can define an object in CoffeeScript  
```coffeescript
cookie = 
    sugar: 5
    flour: 10
    calorieAmount: -> @.sugar * 5 + @.flour * 4
```  
8.) Write some code that changes the background color of your web site every 2 seconds to a random color
```coffeescript
$ ->
  setInterval ->
    color1 = Math.floor Math.random() * 255
    color2 = Math.floor Math.random() * 255
    color3 = Math.floor Math.random() * 255
    $('body').css
    background: "rgb(" + color1 + ", " + color2 + ", " + color3 + ")"
  , 2000
```