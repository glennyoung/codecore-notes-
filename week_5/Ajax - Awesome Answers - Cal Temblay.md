#REFACTORING APP TO USE AJAX


**answers/_form.html.erb**

<%= form_for [@question, @answers], remote: true do |f|
*attaches an event handler that takes the form information and submits and then returns false.*

Now when we click Create Answer, the page does not reload. The app starts a POST and now is sending info as JS, not HTML anymore. The answer wasn’t posted yet until we refresh.. it was created just not updated live yet. So even if your device doesn’t support JS, the site won’t break


**AnswersController**

Select format to send info as. The default is html. This can be useful say we want to use an API that uses JSON then we will want to export the info we collect as JSON.

Whatever we put in the block it will try to read the format, if it’s HTML it will do { } and if it’s JS it will do something different.

````
def create
respond_to do |format|
     if @answer.save
       AnswersMailer.notify_question_owner(@answer).delegate
       format.html { redirect… }
       format.js { render }
     else
       format.html { render “questions/show” }
       format.js { render }
  
     end
end
```

format.js { render } will render create.js.erb which we need to create in views/create.js.erb - since we’re not doing a redirect anymore, we need the view file.

## DISPLAYING ANSWERS

**show.html.erb**

We should create a new partial using the display code for Answers found in show.html.erb:
````
<% @question.answers each do |answer| %> pasted in _answer.html.erb
````

in show.html.erb:
below the display code before the end tag we need to render the answer partial and pass it the @answers:

````
<div class=“answers”>
<% @question.answers.each do |ans| %>
<%= render partial: “/answers/answer”,  locals: { answer: ans} %>
<% end %>
</div>
````

shortcut version: <%= render @question.answers %>

When rendering a partial we don’t need to pass instance variables. using a partial makes our life easier later. When we are rendering all the answers and change/add more, if we use @answer then it will just use the last answer, hence the lack of instance variable used in rendering the partial.


**create.js.erb**

````
$(“.answers”).prepend(“<%= j render partial: '/answers/answer', locals: {answer: @answer} %>);
````

We’re in the Answer create method so we can pull the answer from it.

The “j” escapes javascript, makes it safe, and prevents issues with quotes when getting your data. (rails magix)

Now we need to clear the answer input field after submitting:

````
$(‘#answer_body”).val(“”);
````


## ERROR HANDLING 

Always think about partials when rendering with AJAX. 

***create.js.erb**

````
<% if @answer.errors.any? %>
     $(“.answer-form”).html(“ <%= j render “/answers/form” %>”);
<% else %>
$(“.answers”).prepend(“<%= j render partial: ’/answers/answer’, locals: {answer: @answer} %>);
<% @answer = Answer.new %> 
$(“.answer-form”).html(“<%= j render “/answers/form” %>”);
<% end %>
````
The last render is to display the new answer created, otherwise users would have to refresh the page.

Since we already have a way to handle errors in _form.html.erb all we have to do is re-render this in *show.html.erb*:

````
<div class=“answer-form”>
     <%= render “answers/form %>
</div>
````

## DELETING ANSWERS 

**_answer.html.erb**

We need to add remote: true to pass JS instead of html.

````
<%= link_to “Delete”, question_answer_path(@question, answer), data: {confirm: “Are you sure”}, method: :delete, class: “btn btn-sm btn-danger”, remote: true %>
````

**answers_controller**

We need to handle the redirection after deleting an answer. Do not use redirect_to when passing JS: 

````
def destroy
     @answer.destroy 
     respond_to do |format|
          format.js { render js: “alert(‘deleted’);” }
          format.html { redirect_to question_path(@question) }
     end
end
````

### FADING OUT DELETED QUESTION

**_answer.html.erb **

Wrap entire answer with id so we can target the specific answer
````
<div id=“<%= dom_id(answer) %>”>
````
Now the div an answer is contained in has “id=answer_id” and we can target it like a pro. dom_id is like putting “answer_id”. If we used answer.id we might have conflict because some comments could have the same ID.

**create views/answers/destroy.js.erb**

````
$(“#<%= dom_id(@answer) %>”).fadeOut();
````







