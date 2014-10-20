Let's implement the zipObject function that nables such results
```javascript
zipObject(['fred', 'barney'], [30, 40]);
  => { 'fred': 30, 'barney': 40 } 
  
zipObject([['fred', 30], ['barney', 40]]);
  => { 'fred': 30, 'barney': 40 }
```  
The zipObject creates an object composed from arrays of keys and values. It is provided with either a single two dimensional array, i.e. [[key1, value1], [key2, value2]] or with two arrays, one of keys and one of corresponding values.

If only keys are given, then set the values to undefined.
```javascript
zipObject(['fred', 'barney']);
  => fred: undefined, barney: undefined }
```  
If neither keys nor values are specified, then return {}
```javascript
zipObject()
{}
```

## Solution  
```javascript

function zipObject(keys, values) {
  var returnObject = {};
  if(values){
    for(var i=0; i<keys.length; i++) {
      returnObject[keys[i]] = values[i];
    }
  } else {
    if(keys[0] instanceof Array && keys[1] instanceof Array) {
      for(var i = 0; i < keys.length; i++) {
        returnObject[keys[i][0]] = keys[i][1];
      }
    } else {
      if(typeof keys[1] === 'string') {
        for(var i = 0; i < keys.length; i++) {
          returnObject[keys[i]] = undefined;
        }
      }
    }
  }
  return returnObject;
}




console.log(zipObject(['fred', 'barney'], [30, 40]));
console.log(zipObject([['fred', 30], ['barney', 40]]));
console.log(zipObject(['fred', 'barney']));
```
