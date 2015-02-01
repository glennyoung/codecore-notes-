# Sass  
[Less](http://lesscss.org/) | [Sass](http://sass-lang.com/)(Syntactically Awesome Stylesheets) | [Useful Mixins](http://zerosixthree.se/8-sass-mixins-you-must-have-in-your-toolbox/)  
  
Less and Sass are CSS extention languages.  
  
Try out some online tools to compare [CSS to Less](http://css2less.cc/), and [CSS to Sass](http://css2sass.heroku.com/). There are some [differences between LESS and SASS](https://gist.github.com/chriseppstein/674726), namely [selector inheritance](https://gist.github.com/chriseppstein/674726#selector-inheritance), [conditionals & control structures](https://gist.github.com/chriseppstein/674726#conditionals--control-structures), [namespaces](https://gist.github.com/chriseppstein/674726#namespaces). Both would need to be pre-processed (compiled) to CSS before it will run in your browser. For Rails apps, this happens for us.  
  
## Mixins  
A mixin can let you define, almost like a class.
```scss
/* define a mixin called border radius */
@mixin border-radius($radius) {
  -webkit-border-radius: $radius;
     -moz-border-radius: $radius;
      -ms-border-radius: $radius;
          border-radius: $radius;
}

/* use border-radius and pass it in 10px. */
.box { @include border-radius(10px); }
```  
Use established colors  
```css
.brand-color-borders{
  border-color: $brand-color;
}
```  
Nesting. We can next selectors within selectors, rather than have to chain selectors together and re-declare their values.  
```scss
/* CSS */
#my-section {
  color: #454545;
}

#my-section a {
  color: #232323;
}


/* SCSS */
#my-section {
  color:454545;
  a {
    color:232323;
  }
}
```  
Partials are prefixed with an underscore in SCSS. You can include all those mixins variable you may have defined with @include. For example, we included the border-radius mixin above, like so    
```scss
.box { @include border-radius(10px); }
```  
You can extend a class. For example, you might have btn-success with its own styling, and then extend it, like so  
```css
btn-twitter {
  @extend .btn-success;
  background-color: $twitterBlue;
}
```  
[Extending and Inheritance](http://sass-lang.com/guide#topic-7) is easy and fun.  
```scss
/* You can define some attribute values for a class */
.message {
  border: 1px solid #ccc;
  padding: 10px;
  color: #333;
}

/* then extend the class, and add more attribute values */
.success {
  @extend .message;
  border-color: green;
}

/* Scss is a little bit less writing that Css */
.message, .success, .error, .warning {
  border: 1px solid #cccccc;
  padding: 10px;
  color: #333;
}

/* or is it? */
.success {
  border-color: green;
}
```  
[Operators](http://sass-lang.com/guide#topic-7) are pretty cool, and super easy to use, too!  
```scss
/* see :-) */
.container { width: 100%; }

article[role="main"] {
  float: left;
  width: 600px / 960px * 100%;
}

aside[role="complimentary"] {
  float: right;
  width: 300px / 960px * 100%;
}
```
