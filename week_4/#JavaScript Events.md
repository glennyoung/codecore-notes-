#JavaScript Events  
Events can be clicking your mouse, hovering over something, clicking on something, etc. We can say something like, "when we click this button, show an image pop-up," for example.  
  
Let's look at `$(document).ready(function(){});` for a moment. One solution to have JavaScript execute only when the document is ready, we could put our JavaScript at the end of the document. A more solid way would be to specify that it should run on document ready. The following are equivalent.  
```javascript
$(function(){

});

jQuery(document).ready(function(){

});

$(document).ready(function(){

});
```  
Here's an example of an event (selecting all the elements that have a class called 'my-button'.)
```haml
= link_to "Click Me", "javascript:void(0);", class: "btn btn-info my-button" 


```
```javascript
// one way
var showAlert = function(){
  alert("hey");
};

var docReadyFunc = function() {
  $(".my-button").click(showAlert);
}


// another way
$(document).ready(function() {
  $(".my-button").click(function(){
    alert("hey");
  });
});

```  
jQuery on. We pass in the event on "click", the action to do it on  
```javascript
$(".button-container").on("click", "my-button", function(){
  alert("Hey");
});
```   
Calling a function to show the id of the element that accessed it.    
```javascript
var showAlert = function() {
  alert($(this).attr("id"));
}
```  
jQuery [Events](http://api.jquery.com/category/events/)  
```javascript
$(".my-button").fadeOut();
$(".my-button").fadeIn();
$(".my-button").dblclick();
$(".my-button").focus();
$(".my-button").blur();

$('selector').mouseover(one_function);
$('selector').mouseout(one_function);
$('selector').hover(one_function, two_function);
```  
1.) Write some javascript, that makes the row fade out when you click the delete button.  
```javascript

$(document).on("click", ".btn-remove-block", function() {
  $(this).parents(".well").fadeOut();
});
```  
2.) Create an input field that adds to an unordered list when you hit the enter key.  
```erb
tasks.html.erb
<%= text_field_tag :task, '' %>
<ul class="tasks">
</ul>

```
```javascript
$(document).on("keyup", "#task", function(event) {
  var $li, keycode;
  keycode = event.keyCode ? event.keyCode : event.which;
  console.log(keycode);
  if (keycode === 13) {
    $li = $("<li>", {
      text: $(this).val(),
      "class": "task"
  });
  $("ul.tasks").append($li);
  $(this).val("");
  }
});

```  
3.) Make it so that when you click on a list item, it fades out.  
```javascript
$(document).on("click", ".tasks li", function(){
  $(this).fadeOut();
})
```  
Example of Bubbling  
```javascript
// javascript
$(document).ready(function() {
  $(".box1").click(function() {
    alert("box1");
  });
  $(".box2").click(function() {
    alert("box2");
  });
  $(".box3").click(function() {
    alert("box3");
  });
});
```  
```haml
/ haml
.box1{style: "width: 200px; height: 200px; background-color: red;"}
  .box2{style: "width: 100px; height: 100px; background-color: blue;"}
    .box3{style: "width: 50px; height: 50px; background-color: green;"}
```  
4.) some jQuery to take an input string, and output it reversed. Also, add a function so when the input field is focused, the output string beside it reads "start typing..."      
```javascript
// Javascript
$("#task").on("focus", function() {
  if (!$this).val()) {
      $("#result").text("start typing..");
    }
});

$("#task").on("keyup", function(event) {
  var backward = $(this).val().split("").reverse().join("");
  var keycode = event.keyCode ? event.keyCode : event.which;
  if (keycode === 9){
    return;
  }
  $("#result").text(backward);
});
```  
Some last things.  
```javascript
e.preventDefault();

// vs
e.stopPropogation();

// vs
return false;
```  
Let's say I have an anchor `<a href="http://cnn.com"></a>` it's going to go to cnn.com when I click on it. But if we add an event to it, such as `event.preventDefault` it will prevent to default to go to CNN.com. Same thing if you submit a form. A form's default is to submit. This will prevent that.  
  
If you `return false`, it will stopPropogation, and preventDefault for the event. For example the following should execute all three if you click the small box whith is contains witin the other two.  
```javascript
// javascript
$(document).ready(function() {
  $(".box1").click(function() {
    alert("box1");
  });
  $(".box2").click(function() {
    alert("box2");
    return false;                  // returning false here will prevent any further events from bubbling up
  });
  $(".box3").click(function() {
    alert("box3");
  });
});
```  
5.) Create a form with 3 fields: 'first name', 'last name', and 'email', that displays the values below when 'save' is clicked.
```javascript
$(document).on("submit", "#personal-info", function(event){
  var result;
  result = "First Name: " + $('#First_Name').val() + " " + "Last Name: " + $('#last_name').val() + "Email: " + $('#Email');
  $('#display').text(result);
  return false;
});
```
```haml
= form_tag "/" id: "personal-info" do
  = text_field_tag "First Name", "", placeholder: "First Name"
  = text_field_tag "Last Name", "", placeholder: "Last Name"
  = text_field_tag "Email", "", placeholder: "Email"
  = submit_tag "Submit"
```  
6.) Make a button jump randomly around a page. It only stops if you are able to click it, and then it displays "SUCCESS!!!"  
```javascript
// javascript
```
  
