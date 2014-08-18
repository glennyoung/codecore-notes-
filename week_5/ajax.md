# Ajax  
  
To make your answer form on the question show view use AJAX, open up simply add `remote: true` in the `form_for` line  
  
```haml
/ app/questions/views/show.html.haml
= form_for @answer, url: question_answers_path(@question), remote: true do |f|
  = f.text_area :body, class: "form-control"
  %br
  = f.submit "Submit an answer", class: "btn btn-primary"
  %hr 

/ We can replace the following 
-#  -@question.answers.each do |answer|
-#    = render "/answers/answer", answer: answer

/ With this (because we will tell our answers_controller create method to render some javascript): 
.answers= render @answers 
 ```  
 Then, inside the answers controller, let's look at the create method. We can add a respond_to to give some format options.  
 ```ruby
 # app/controllers/answers_controller.rb
 
 # ...
 
   def create
    #@question = Question.find params[:question_id]
    @answer = @question.answers.new(answer_attributes)
    @answer.user_id = current_user
    respond_to do |format|
      if @answer.save
        # AnswerMailer.notify_question_owner(@answer).deliver
        AnswerMailer.delay.notify_question_owner(@answer)
        format.html { redirect_to @question, notice: "Answer created successfully" }
        format.js   {render}  # here rails will look for a view of the name of the method 'create.js.haml'
      else
        format.html { render "questions/show" }
        format.js   {render js: "alert('ERROR');"}
      end
    end
  end
  
# ...

```  
Now, if we change the line `format.js {render js: "alert('created');"}` to just `format.js { render }` it will look for a create.js file. Let's make one.  
```haml
/ app/views/answers/create.js.haml
$('#answer_body').val(""); 
  $('.answers').prepend("#{j render 'answer', answer: @answer}");
  $("##{dom_id(@answer)}").hide().fadeIn(500);
```  
If we want to prepend an answer to a div of class 'answers', we should make sure this exists in our questions show view.  
```haml
/ app/views/questions/show.html.haml
.answers
  -@question.answers.each do |answer|
    = render "/answers/answer", answer: answer
```  
Add `.well{id: dom_id(answer)}` to the answers partial so each answer has a dom id.  
```
/ app/views/answers/_answer.html.haml
.well{id: dom_id(answer)}  
  .row
    .col-sm-8.col-md-8.col-xs-8
      %p= answer.body      
      %p Created on #{formatted_date(answer.created_at)}
    .col-sm-4.col-md-4.col-xs-4
      .pull-right= button_to "Delete", question_answer_path(@question, answer), method: :delete, confirm: "Are you sure you want to delete this?", class: "btn btn-danger", remote: true
``` 