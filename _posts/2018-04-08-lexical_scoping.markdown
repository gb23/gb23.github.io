---
layout: post
title:      "Lexical Scoping"
date:       2018-04-08 22:38:09 +0000
permalink:  lexical_scoping
---


First things first -- **function declaration** vs. **function invocation**:
- Function Declaration.
    - This is where keyword `function` is usually typed.  Following this keyword is the function name with parenthesis containing any (or none at all) parameters.  Next is the opening and closing set of curly brackets.  Within these brackets is the code that defines what the function does.
- Function invocation.
    - This is where we take a function handle, usually the function name itself, and apply the invocation operator: `()`, with any necessary parameters.


## What matters for lexical scoping: the function **declaration**.
For the sake of explanation, let's pretend javascript functions have thoughts, desires, and human characteristics.  Javascript functions *do not care* where they are **invoked**.  Javascript functions *care* where they are declared.  

When the programmer declares a new function, the function is *born* into the world.  The *world* is what surrounding the function -- the outer scope.  The outer environment gets stored in the function's scope chain.  This is called lexical scoping.  

Let's look at some examples.

### Example A:
```
const myVar = 'Foo';
 
function first () {
	console.log('Inside first()');
	console.log('myVar is currently equal to:', myVar);
}
 
function second () {
	const myVar = 'Bar';
	first(); 
}

second();
```
Let's ignore what the functions do for a moment.  This will declutter the code, so we can focus on lexical scoping.
```
const myVar = 'Foo';
 
function first () {...}
 
function second () {... first() ...}

second(); //funtion invocation
```
Much cleaner, and it still gives us important information about scoping.  On the last line, we **invoke** the function named `second`.  The invocation operator applied to the function name `second` means we execute the code inside the curly bracket block defining `second`.  Here, we will invoke the function `first`.  We then execute the code defining the function `first`.  Cool.  Now that we've looked at the blueprint of the code, let's expand the code back to include everything.
```
const myVar = 'Foo';
 
function first () {
	console.log('Inside first()');
	console.log('myVar is currently equal to:', myVar);
}
 
function second () {
	const myVar = 'Bar';
	first(); 
}

second();
```
We know this code.  Invoke funcion second.  Execute code of `second()`.  Here we invoke the function `first`.  We execute the code of `first()`. What is the value of myVar we print? 

The trap: forgetting that Javascript functions *do not care* where they are **invoked**.  Javascript functions *care* where they are declared.  When the scope chain for the function `first` is created, it looks at the surrounding environment of the function's declaration.  In it's outer scope, `myVar` is assigned the string `'Foo'`.  Inside `first` there is no variable `myVar`, so we go up the scope chain to the global `myVar` of 'Foo'.  


**We do not care that `myVar = 'Bar'` inside the function `second`.  *THIS IS THE TAKEHOME MESSAGE FROM THIS EXAMPLE***

Function output:
```
// LOG: Inside first()
// LOG: myVar is currently equal to: Foo
```

### Example B:

```
const myVar = 'Foo';
 
function second () {
	
	function first () {
		console.log('Inside first()');
		console.log('myVar is currently equal to:', myVar);
	}
 
	const myVar = 'Bar';
 
	first();
}

second();
```
Similar to Example A, but slightly different.  Function `first` is now *born* (DECLARED) in a different scope environment: the scope of the function `second`.  Any idea what the value of `myVar` will be when it is logged inside of the function `first`?

So, we invoke the function `second`.  The body of the function `second` is executed.  We invoke the function `first`.  The value of the variable `myVar` is needed.  Inside of the function `first`, there is no variable `myVar` so we go up the scope chain.  We see `myVar = 'Bar'`.  Boom. That's the value.  We log it.

**We do not care that `myVar = 'Foo'` in the global scope.  It is too far down the scope chain.  *THE TAKEHOME MESSAGE IS THAT WE LOG THE FIRST VALUE OF MYVAR THAT WE FIND IN THE SCOPE CHAIN***

Function Output:
```
// LOG: Inside first()
// LOG: myVar is currently equal to: Bar
```
