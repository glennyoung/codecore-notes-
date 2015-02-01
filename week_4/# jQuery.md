# jQuery  
[Prop vs Attr](http://stackoverflow.com/questions/5874652/prop-vs-attr) |   
  
jQuery is a library that helps us use JavaScript. With jQuery, we can use `$` to say we want to use jQuery.  
```javascript
$("p")[0]                   // select an index by element tag
$(".btn")                   // select by class name
$("div.question-listing")   // find an element with a class name <a class="btn">example</a>
$("a.btn")

$("a .btn")                 // this is a class within an element <a><button class=btn"><button></a>
$("a.first .btn")           // find an element with the class "btn" within "a" elements of the class "first"

$("div.btn")
$("div .btn")
```  
We can add a div around our questions in the index view to give a dom id  
```haml
%h1 Listing All Questions
= image_tag "drewbro.jpg", class: "thumbnail"
%br/
= link_to "Create a New Question", new_question_path
- @questions.each do |question|
  %div{:class => dom_id(question)}
    %h2= link_to question.title, question_path(question)
    %p= question.description
    %p
      <Created On: #{formatted_date(question.created_at)}
  %hr
```  
Find more things in jQuery  
```javascript
$("#question_14")               // select a particular id
$("div#question_14")            // select an id within an element
$("[href]")                     // select all elements with an attribute
$("[href='/']")                 // select all elements with an attribute matching the value /

$("[name='question[title]']")    // select a form input with the name question_title
                                 // element = {href: "/", name: "abc"}
                                 // element["href"] 
```  
Special Selectors  
```javascript
$("h2:even")                    // select all even h2 elements
$("h2:odd")                     // select all odd h2 elements

$("form :text")                 // select all elements within form elements which are text
$("input:text")                 // select all input elements that are text
$("input[type='text']")
```  
1.) Find out how to select all checked checkboxes in a form with id new-question  
```javascript
$("form#new_question").is(':checked')
$("new_question :checked")

$("new_question input:checkbox:checked")
$("#new_question [type='checkbox']:checked")  // looking for an element with id of 'new_question' then
                                              // inside it an element of type 'checkbox' that is 'checked'
$("#new_question [type='checkbox'][name='c']:checked")
```
2.) Given a ul with a class 'listing' and many li's, select all the 'a' elements inside even li elements  
```javascript
$("ul.listing li:even a")
$("ul.listing > li:even a") 

// Traverse the tree
$("h2").has("p")                    // select all h2s that have "p" elements
$("h2").first()
$("h2").last()
$("h2").siblings()                  // select all siblings of an element
$("h2").siblings("p.description")   // select all siblings of an element which p with the class "description"

$("#question_15")
$("#question_15").children()        // select all children of an element with a specified id

$("h2").children("[href='/questions/14']") // select all children of an h2 element with the attribute 'href' that has a value '/questions/14'

$("#question_15").parents()          // traverses up the tree
```  
Manipulating
```javascript
$("question_15").show()
$("question_15").toggle()
$("question_15").parents(".container").show()

$("question_15").prop("style")      // the property is whatever is being stored in the background
$("question_15").attr("style")      // and this actually displays it
$("question_15").prop("id")
$("question_15").attr("id")

$("question_15").prop("tagName")
> "DIV"
$("question_15").attr("tagName")
> undefined
$("question_15").attr("id", "abc")

$("h2 a").addClass("btn btn-primary")         // add the class btn btn-primary to all h2 elements
$("h2 a").removeClass("btn btn-primary")
$("h2 a:even").toggleClass("btn btn-primary")      // add or remove class

$("h2 a").remove()                            // removes the a
$("h2 a").hide()                              // hides the a elements within h2's, but can still select them
$("h2 a").show()                              
$("h2 a").empty()


myContent = $(".container").html();
$(".container").emtpy();
$(".container").html(myContent);

$h2s = $("h2");                    // convention in jQuery is to define jQuery variables with a $
$p = $("<p></p>");                 // create a p element in jquery
$p = $("<p></p>", {text: "Hello CodeCore"});

$p.appendTo("h2");                 // put $p inside each h2
$("h2").append($p);
$("h2").prepend($p)
$("h2").before($p);
$("h2").after($p);

$("h2").html("<a> Hello there</a>");
$("h2").replaceWith("<a> Hello there</a>");
$("h2").text("<a> Hello there</a>");  // escapes the html tags

$("h2").css("font-size", "28px");     // dynamically set a css property
$("h2").css("background", "#444");

$("h2").each(function(){ console.log($(this).html()) });



$elements = $("h2");
$elements[0];
$elements[0].css("background", "red");      // this will not work
$(elements[0]).css("background", "red");    // this will work
```  
3.) Write code to increase all H1 element font size by 1
```javascript
// note
Number("12px");
> NaN
parseInt("12px");
> 12


$("h1").each(function() {
  var fontSize = parseInt($(this).css("font-size"));
  fontSize++;
  $(this).css("font-size", fontSize + "px");
});

```  
4.) Write a function that takes in a number n and then generates a ul element with n number of li's inside it.  
  The function should then put hte ul at the end of the body.  
```javascript
var undorderedListGenerator = function(n) {
  var $ul = $("<ul></ul>");
  for(var i=0; i < n; i++) {
    var $li = $('<li>', {text: "Element" + i});
    $ul.append($li);
  }
  $('body').append($ul);
}
undorderedListGenerator(10);
```  
5.) Write code that loops through all the h2 elements that contain 'abc' text and prints the length of text inside each one of those h2 elements.  
```javascript
// log length in the console
$("h2:contains('abc')").each(function(){
  console.log($(this).text().length);
});

// append text to the dom
$h2 = $("h2:contains('abc')");
$h2.each(function(){
  $(this).append(" " + $(this).text().length);
});

```  
6.) Write code that capitalizes all the text inside "a" elements that are inside "h2" elements  
```javascript
// css method
$('h2 a').css("text-transform", "capitalize"); 

// all caps
$('h2 a').each(function(){
  $(this).text( $(this).text().toUpperCase() );
});

// champion answer
$('h2').each(function(){
  var capitalizedText = $(this).text().charAt(0).toUpperCase() + $(this).slice(1);
  $(this).text(capitalizedText);
});

```
7.) Find a random H2 and remove it from the DOM
```javascript
var h2Killer = function(){
  $h2 = $('h2')
  var randomNumber = Math.floor((Math.random()*$h2.length));
  try {
    $h2[randomNumber].remove();
  }
  catch (e) {
    console.log("No more fun, because: " + e);
  } 
}
```
