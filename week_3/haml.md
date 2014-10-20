# HAML  
Hey HAML, why you so nice?  
  
Link: [HTML2HAML](http://html2haml.heroku.com/) | [Styleguide](https://github.com/dbooom/styleguide/blob/master/_styleguide.haml) | [HAML Cheetsheet](http://www.cheatography.com/specialbrand/cheat-sheets/haml/)       

HTML  
```html
<!DOCTYPE html>

<html>

  <head>

    <meta charset="utf-8">

    <title>My Super Awesome Page</title>
  
    <meta keywords="">

    <meta discription="">

    <link rel="stylesheet" href="assets/stylesheets/styles.css">

    <script src="assets/javascripts/myscripts.jss"></script>

  </head>
  <body>

  <h1>This is cool</h1>

  <p>This paragraph is actually about everything that isn't. That's right! It's about nothing :)</p>

  </body>

</html>
```  
Haml  
```haml
!!!
%html
  %head
    %meta{charset: "utf-8"}/
    %title My Super Awesome Page
    %meta{keywords: ""}/
    %meta{discription: ""}/
    %link{href: "assets/stylesheets/styles.css", rel: "stylesheet"}/
    %script{src: "assets/javascripts/myscripts.jss"}
  %body
    %h1 This is cool
    %p This paragraph is actually about everything that isn't. That's right! It's about nothing :)
```  