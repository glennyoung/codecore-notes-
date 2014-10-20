# JavaScript 2  
How do I specify the typeof a variable in JavaScript?  
```javascript
var a = "Hello CodeCore";
typof a;

var _abc = "whiiirld";
var $lol = "there";
var 123abd = "hey";                         // will fail
var _123abd = "hey";

var b = [_123abd, $lol, _abc];

if(0) { console.log("Hello");}              // evaluates as falsy
if(1) { console.log("Hello");}              // evaluates as truthy

if(undefined) { console.log("hey there") }  // falsy

x = 0;
if(x) { console.log("hey there") }          // need to have something defined as x

if(x === 0) { console.log("hey there") }    // always use triple equals

var a = "1";
var b = a + 1;
b                                           // "1" + "1" is concatenating 2 strings "11"

// Creating Object in JavaScript

var myObject = {name: "Drew", city: "Vancouver"}; 

myObject.name
var myObject = {name: "Drew", city: "Vancouver", info: function() { return "This is " + this.name + " from " + this.city}};
myObject.info();                            // call the info function

for(var property in myObject) {             // loop through each property in myObject
  console.log(property);
};

for(var property in myObject) {             // loop through each key, value in myObject
  console.log(myObject[property] + ": " + property);
};

Object.hasOwnProperty("name");              // truthy
Object.hasOwnProperty("abcbaksd");          // falsy

// Build an onject cookie. Give it two attributes: Sugar amount & flour amount

cookie = { sugarAmount: 5, flourAmount: 10 };

// Write a function for the cookie object that returns the amount of calories

parseInt("123");
Number(123);

Number("f", 16);
parseInt("f", 15);

cookie.calories = function() { return this.sugarAmount * 4 + this.flourAmount * 5; };
cookie.calories();

cookie = { 
  sugarAmount: 5, 
  flourAmount: 10,
  calories: function() { 
    return this.sugarAmount * 4 + this.flourAmount * 5; 
  }
};

// What if I want to define a Cookie class?
// In the google javascript console, use ctrl-enter to not submit.
function Cookie(sugarAmount, flourAmount) { 
  this.sugarAmount = sugarAmount;
  this.flourAmount = flourAmount;
  this.calories = function() { return this.sugarAmount * 4 + this.flourAmount * 5 }
}


cookie = new Cookie(10, 5);                   // instantiate a variable cookie as a new Cookie object
cookie.calories();                            // call the calories function

// Prototype Functions
Cookie.prototype.display = function() { console.log("this is a cookie with: " + this.sugarAmount + " g of sugar"); };
cookie.display();
Cookie.prototype.bakingMethod = "High heat";  // pass in a common attribute

// Build an onject: shuffler. 
// Let it have a function generate that takes a number x and then generates and array from 1 to x. 
//Build a function shulle that returns a shuffled version of the array.

function Shuffler(x) {
  this.array = [];
  for(var i = 1; i <= x; i++) {
    this.array.push(i);
  }
  this.shuffle = function() {
    for(var i = 0; i < this.array.length; i++) {
      var newIndex = Math.floor(Math.random() * this.array.length);
      oldValue = this.array[i];
      this.array[i] = this.array[newIndex];
      this.array[newIndex = oldValue];
    }
    return this.array;
  }
}

// Actually create a function called generate

function Shuffler(x) {
  this.x = x;
  this.array = [];
  
  this.generate = function() {
    for(var i = 1; i <= this.x; i++) {
      this.array.push(i);
    }
    return this;
  };
    
  this.shuffle = function() {
    for(var i = 0; i < this.array.length; i++) {
      var newIndex = Math.floor(Math.random() * this.array.length);
      var oldValue = this.array[i];
      this.array[i] = this.array[newIndex];
      this.array[newIndex] = oldValue;
    }
    
    return this.array;
  }

}


shuffler = new Shuffler(25);
shuffler.generate().shuffle();
```