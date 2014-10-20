# JavaScript Selectors  
```javascript

<script type="text/javascript" language="javascript">

</script>
```  
## Content vs Behavior  
The content is in my HTML files, the behavior is in my javascript files. You can just put javascript files in your javascripts directory (app/assets/javascripts/myjavascript.js.coffee)  
  
```javascript
h1 = document.getElementsByTagName("h1")
h1 = document.getElementsByTagName("h1")[0]

div = document.getElementsByTagName("div")

// create a fragment of code
var docFrag = document.createDocumentFragment("small");
var docFrag = document.createDocumentFragment();
var small = document.createElement("small");
small.textContent = "Hello CodeCore";
docFrag.appendChild(small);
h1.appendChild(docFrag);

h1.addEventListener("click", function(){ alter("click click") });

// there be lots o' selectors yo
document.getElementById
document.getElementsByClassName
document.getElementsByTagName

document.getElementsByTagName("small");

// looping through an array using forEach
[1,2,3,4].forEach(function(x) { console.log(x) });

// this does the same thing basically
array = [1,2,3,4]
for(var i = 0; i < array.length; i++) {
  console.log(array[i]);
}
```  