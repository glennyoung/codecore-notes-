# More Javascript  
[github repo](https://github.com/tkbeili/jquery_drills) |    
Form input validations.  
1.) Create an input form with two fields. One for email, and one for password. If the email is not a valid format, show an x, otherwise show a check. If the password is fewer than 8 characters, show an x, otherwise show a check. Perform the check on focus out.  
```javascript

```  
2.) Use draggable and droppable. Make 4 boxes: 1 large and 3 small. The small boxes should be draggable and droppable onto the large box.  
```javascript
$ ->
  $(".box").draggable()
  $("#drop-box").droppable
    drop: (event, ui)->
      alert("You dropped an Item with id: " + $(ui.draggable).attr("id"));
```   
3.) Make a button that alternates between three classes on click: "btn-danger", "btn-primary", and "btn-info".  
```javascript
$ ->
  classes = [""]
  currentIndex = 0
  $(".btn").on "click", ->
    $(@).toggleClass(classes[currentIndex])
    currentIndex = if currentIndx >= (classes.length-1) then 0 else currentIndex + 1
    $(@).addClass classes[currentIndex]

```  
4.) Create a checkbox list that when you check items on the list, they gain a strike-through.  
```haml
/ haml to generate the checkboxes
%ul{style: "" }
  - 10.times do |x|
    %li.checkbox
      %label
        = check_box_tag "box#{x}", "abc", nil, class: "chkbox"
        = "Check Box #{x}"
```    
```javascript
$('.chkbox').on "change", ->
  if @.checked
    $(@).parent().addClass("checked")
  else
    $(@).parent().removeClass("checked")
```  
```css
.checked { text-decoration: line-through; }
```
5.) Using the list you created in the previous exercise, add the functionality so that items that are checked move to a "completed items" div  
```javascript

$ ->
  $(".chkbox").on "change", ->
    if @.checked
      $(@).parent().addClass('checked')
      $li = $(@).parents "li"
      $li.fadeOut 500,
        $(@).appendTo ".completed"
        $(@).fadeIn 300
    else
      $(@).parent().removeClass("checked")
      $li = $(@).parents "li"
      li.fadeOut 300, ->
        $(@).appendTo ".pending"
        $(@).fadeIn 300
```