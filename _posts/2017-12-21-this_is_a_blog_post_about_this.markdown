---
layout: post
title:      "This is a blog post about _this_."
date:       2017-12-22 02:38:22 +0000
permalink:  this_is_a_blog_post_about_this
---


## The Rule to Remember (in pseudocode):
```javascript
if ('this' is referenced directly inside a METHOD){
    'this' = object that received the method call
} else {
    'this' = global
}
```
Another way of saying The Rule to Remember: `this` changes depending on whether it is referenced from inside a METHOD versus inside a FUNCTION. `this` inside a method equals the object.   `this` inside  a function equals global. So what's the difference between a method and a function?

### Definitions:
* A JavaScript **method** is a property of an object, and this property points to a **function**.  For example:  
```javascript
const teslaCar = {
    hitHorn: function(){
        console.log('honk');
    }
};
```
Here we have an object _teslaCar_ which has a _hitHorn_ property.  This property points to a function, so this property, _hitHorn_ is a **method**!  You can see that a method is a more specific type of **function**. A **function** that is not a **method** would not be part of an object:
```javascript
const hitHorn = function(){
    console.log('honk')
} 
```
In the first example we have to invoke hitHorn on a teslaCar object:
```javascript
    teslaCar.hitHorn()
    //honk!
    //=> undefined
```
So hitHorn is a **method**!

In the second example we cannot invoke hitHorn on a defined object.  We simply just invoke it:
```javascript
    hitHorn()
    //honk!
    //=> undefined
```
So hitHorn is a **function**!

## Functions vs Methods as it Pertains to _this_
Let's change our object definition ever so slightly, using _this_:
```javascript
const model = "camry";

const teslaCar = {
  model: "model3",
  hitHorn:  function(){
    console.log(this);
    return this.model + " goes honk!";
  }
};
```
Before freaking out about the _this_, just remember The Rule To Remember.  Is hitHorn a method or just a function?  It's a method because it is invoked on the tesla car object. The Rule To Remember says that the _this_ inside of a method is the object that has the method.  So teslaCar is _this_! 
```javascript
teslaCar.hitHorn();
{ model: 'model3', hitHorn: [Function] }
//=> 'model3 goes honk!'
```
What is logged in the console? An object with a property `model` set to 'model3' and a `hitHorn` property that points to a function.  Hey, that's a teslaCar object! So, _this_ is teslaCar.

Note that in the return statement, `this.model` refers to teslaCar's model, NOT the global model of camry.

Let's add a line of code at the end of our example:
```javascript
const model = "camry";

const teslaCar = {
  model: "model3",
  hitHorn:  function(){
    console.log(this);
    return this.model + " goes honk!";
  }
};

const makeNoise = teslaCar.hitHorn;
```
Now observe the following:
```javascript
makeNoise();
//Window {...}
// =>'camry goes honk!'
```
Interesting!  We are console.logging a value of `Window{...}` and returning `'camry goes honk!'` Why?
In the line `const makeNoise = teslaCar.hitHorn;`, we are saying "create a reference to the same function to which `teslaCar.hitHorn` refers.  
That is, this function now has two references: hitHorn and makeNoise.
The difference (remember, remember The Rule To Remember) is that hitHorn is a property of our object (and thus a **method**) whereas makeNoise is not; it is just a **function**.  Put another way, hitHorn is invoked with our object (`teslaCar.hitHorn()`) whereas makeNoise is invoked without it (`makeNoise()`). `makeNoise()` knows nothing about the teslaCar object. So, the _this_ in our function makeNoise() refers globally.  In the context of the browser, `Window` is the global object.  When we call `this.model` in that function, _this_ is global, so it looks for global definitions of model.  It finds `const model = "camry"`, so "camry goes honk!" is returned.

## Functions within Functions
So you may be thinking oh I see the pattern: with [object instance].[method] any `this` refer to the object.  Not so fast.  Remember the Rule to Remember.
```javascript
const teslaCar = {
  hitHorn:  function hittingTheHorn(){
    function handOnTheHorn(){
      return this;
    }
    return handOnTheHorn();
  }
};

teslaCar.hitHorn();
```
Let's break this down: We have a teslaCar object with a hitHorn property that references a function. So `hitHorn` is a method! Correct.  But where is the _this_? When `teslaCar.hitHorn()` is invoked, `hittingTheHorn` is executed.  There is the `handOnTheHorn` declaration but nothing to execute until the last line `return handOnTheHorn();`.  At this point handOnTheHorn() is invoked and we reach `this`.  What is `this`?  The  `this` is inside of `handOnTheHorn`, so we need to remember The Rule and ask ourselves: is handOnTheHorn a method or a function? It is a function.  To be a method, there would have to be a reference to this function from the object teslaCar and there is not.  It is essentially 'hidden' away from the object because it is inside a function.  `this` is returned from `handOnTheHorn` and then that is returned from `hitHorn` and thus our console gives us a value of `Window {...}` as the result because the `this` was inside of a function (not a method) and was thus global.

## _this_ in callbacks

```javascript
const cars = ['modelS', 'modelX', 'model3', 'camry', 'accord', 'civic']

cars.filter(function(car){
    console.log(this);
    return car.charAt(0) === 'm'
})
```
We have this filter method and we want to take an array of car names and generate an array of tesla cars (the ones that start with 'm').  This works fine but what is the value of this? When I run this code I get: 
```javascript
Window{ … }
Window{ … }
Window{ … }
Window{ … }
Window{ … }
Window{ … }
//=>[ 'modelS','modelX','model3']
```
With each iteratation of the array, the _this_ is global, hence we are seeing window.  But why?  Think about how callbacks work! We are dealing with an array object which has a reference to 
[Array.prototype.filter()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) method (yes method because array is the object and it has a property which references this filter function). Lets look at how this filter code would look like:
```javascript
Array.prototype.filter( our callback definition is a param ){ 
    //do filtery things
    //...
    //Our callback definition is placed somethere in here for use:
    function(car){
        console.log(this);
        return car.charAt(0) === 'm'
    }
    //finish doing filtery things
    //...
}
```
Hey look at that, a function inside of a method, just like before!  Just like before, because the `this` is inside of a function, not a method, it is global.  Hence when we `console.log(this)` we will get `Window{ ... }`.


## Using `bind()` in callbacks to change value of `this`
Let's add bind() to a callback and see how that changes the logged value of `this`.
Without the bind(), we run into the same issues with `this` is a callback:
```javascript
class Driver {
    constructor(name, favoriteCarCompany){
        this.name = name;
        this.favoriteCarCompany = favoriteCarCompany;
    }
    favoriteCarCompanyMatches(cars){
        // here, `this` is the Driver instance
        return cars.filter(function(car){
            // here, `this` is global
            return car == this.favoriteCarCompany
        });
    }
}

const greg = new Driver('Greg', 'Tesla')
greg.favoriteCarCompanyMatches(['Mercedes', 'BMW', 'Tesla', 'Lexus'])
```
Running this code gives us:
```
Uncaught TypeError: Cannot read property 'favoriteCarCompany' of undefined
```
Why?  It's saying you can't call `this.favoriteCarCompany` because `this` is undefined.  It's global.  Remember our previous example with the filter method and the issue we had with the `this` inside of the callback?  Same thing here. 

`favoriteCarCompanyMatches()` is a method on the driver instance object, so any `this` value directly mentioned inside that method would be the driver instance.  But, once we go inside the callback, we are now inside of a function of the filter method.  What we need to do is bring the `this` that is available direcly inside favoriteCarCompanyMatches and "bind" it to the callback. Decluttered it looks something like this:
```javascript
class Driver {
    ...
    favoriteCarCompanyMatches(cars){
        // here, `this` is the Driver instance
        someObject.someMethod(  function(car){...}.bind(this)   );  
    }
}
```
The callback function is declared when the favoriteCarCompanyMatches method is invoked. Upon invokation of this method, `this` equals the driver instance receiving the method call, and we bind the callback function to that diver instance. Then, from inside the filter method, the callback function is invoked. Because we bound the driver instance to the callback we do not have to deal with the normal case where `this` would be global. In the example above the scope area of ```...``` would have `this` equal to the driver instance.  Applying this to the entire code results in:
```javascript
class Driver {
    constructor(name, favoriteCarCompany){
        this.name = name;
        this.favoriteCarCompany = favoriteCarCompany;
    }
    favoriteCarCompanyMatches(cars){
        // here, `this` is the Driver instance
        return cars.filter(function(car){
            // here, `this` is now the Driver instance
            return car == this.favoriteCarCompany
        }.bind(this));
    }
}

const greg = new Driver('Greg', 'Tesla')
greg.favoriteCarCompanyMatches(['Mercedes', 'BMW', 'Tesla', 'Lexus'])
```
with the output:
```javascript
["Tesla"]
```
So _that_ was _this_!

