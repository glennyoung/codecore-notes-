#JavaScript Primer  
When we practice JavaScript, there's a great tool call [repl.it](http://repl.it).  

In the google browser, if you open your console (cmd-opt-j), you can access the javascript console for Chrome!.  
***note***: Mozilla has great [JavaScript docs](https://developer.mozilla.org/en/docs/Web/JavaScript)!  
```javascript
var hello = "Hello World";    // in JavaScript we all a variable a var

alert(hello);                 // alert gives a little alert popup with the specified message.

confirm("Are you sure?");     // confirm dialog

var greetings = "Hello CodeCore";
console.log(greetings);       // log something to the console.

greeting = "Hey, there whiiiiiiirld.";    // if you don't put var before the variable, it will
                                          // put it in a global scope.
                                          
var i = 10;                               // we don't explicitly specify types
console.log(i / 3);                    
console.log(i % 3);

var myNumber = "1000";                    // convention in JavaScript is to name variables in camelcase starting with lower case.

console.log(Number(myNumber)/100);        // end a line with a semi-colon.
console.log(parseInt(myNumber)/100);
console.log(parseFloat(myNumber)/0);      // gives us infinity

Infinity + 1
Inifinity
Infinity - Infinity

1 == "1"                                  // true - double equals evaluates equality
1 === "1"                                 // false - triple equals evaluates equivalency
1 === 1                                   // true - true equivalence

Math.random()                             // generates a random number between 0 and 1
Math.random() * 100                       // generates a random number between 0 and 100

Math.floor() 
Math.round()

var a = "Hello world";
a.substring(0, 2);                        // Start from 'first', go 'second' number of letters.

var myFunction = function(){              // define a variable as a function.
  console.log("I'm inside the function.");
};

function myFunction() {                   // another way to define a function.
  console.log("I'm inside the function.");
};

myFunction();                             // call the function.

function myFunction(x, y){                    // another way to define a function.
  console.log("Hi " + x + ". I'm " + y + "!"); // there's no string interpolation in JavaScript use + to concatenate
};

function myFunction(x, y){
  console.log("I'm inside the function" + y + y);
  return x + y;
 };
 
var myArray = [1, 2, true, "Hello CodeCore"];
console.log(myArray[2]);

console.log(myArray.length);

var myMultiArray = [[2, 3, 4], [3, 4, 5, 6, 7]];

for(i = 0; i < myArray.length; i++) {
  var x = "Hello";                          // local loop variable
  console.log(myArray[i]);
}

var x = 1;

while(x < 100) {
  console.log(x);
  x++;
}


while(x != 50) {
  console.log(x);
  x = Math.floor(Math.random() * 50);
}

var myObject = {};
myObject.name = "Tam";
myObject["age"] = 30;
console.log(myObject.name);


{user: {firstName: "Tam"}}

var personalInfo = {
  firstName: "Tam", 
  lastName: "Kbeili",
  languages: ["Ruby", "Javascript"],
  fullName: function() {
    return this.firstName + " " + this.lastName;
  }
};

console.log(personalInfo.fullName());
```