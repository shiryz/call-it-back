# Call it Back

This Readme tries to better introduce the concept of callbacks to those new to it, it assumes you have a basic understanding of JavaScript and know what functions are.
To understand it better, I would recommend trying the examples out yourself and even changing some things to get a better sense of what's happening when and how.

## Functions all the way

#### Functions can do it all 

- A function is a first-class object.
- has methods, check https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/prototype#Methods
- has properties, check https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/prototype#Properties 
- can be assigned to a variable, 
```js
function foo() {...}
var someFunction = foo;
```
or 
```js
var someFunction = function() {...};
```
Notice the second definition uses anonymous functions.
- And a lot more...

#### What else?

- what are these lines doing? can we assign a function to an object?

```js
var obj = {1: function(){ console.log("The first property");}, 2: "string"};
obj[1]();
```
Answer: Yes you can, if you run these lines you'll get "The first property" logged to your console.
Notice that we invoke the function by adding parentheses to `obj[1]`, without them all we'll get is the function itself (lines of code, that mean nothing if not executed!).

- And arrays? can a function be a value in an array?
Yes! it can.
what is this code doing?
```js
var functionsArray = [
function(){console.log("I'm the first value in the array");}, 
function(){console.log("I'm the second");}, 
function(){console.log("Hello I'm third");}];

functionsArray.forEach(function(fun){
 fun();
});
```
`fun()` will execute all the functions in the array.


## So, what's a callback then?

A callback function, also known as a higher-order function, is a function that is passed to another function.
We already know that a function can do anything any object can do, that also means they can be passed around as arguments, for example:
```js
function foo (myCallBack){
  myCallBack();
}
```
foo expects one argument as an input, this argument needs to be a function, since we call it.
trying to execute this line 
```js
foo("a string");
``` 
would error with the following: `TypeError: myCallBack is not a function`, this is happening because we use the argument myCallback as a function.

_How do I send a function though_ :thinking:?

As mentioned before, functions are objects in every way, so a function definition can be passed as an argument too:
```js
function theCallBack(){
 console.log("Jack, come back");
}
foo(theCallBack);
```
Here we can see `theCallBack` passed to foo as an argument and as expected this will log `Jack, come back`.

Another way you can pass a function is by using anonymous functions (a function doesn't have to have a name, same if we pass `3` instead of `x` with a value of 3), this is how it would look like:
```js
foo(function(){
  console.log("expected a function, got an anonymous one :|");
});
```

## Funception 

We're going to back up a bit to understand function evaluation, what happens if a function is passed as an argument to another function and that is passed to another function as an argument and so on...

__What's happening here?__ 
```js
foo1(foo2(foo3()));
```
What's foo1's input? in short it's what is being evaluated in its parentheses or in this case the result of `foo2(foo3())`.
Same goes for foo2's input which is the result of calling foo3().

Let's try to analyze what javaScript's interpreter is doing:
```js
function foo1(number){
 return number*3;
}
function foo2(string){
 return Number(string);
}
function foo3(){
 return "1";
}

foo1(foo2(foo3()));
```
If you're familiar with the call stack, then you know that `foo3()` will be called first because it will end up on top of the stack, if not then you should know that evaluation starts from the right most call, which means we'll execute `foo3` first returning `"1"` as an argument to `foo2` then `foo2` will return `1` as a number and lastly `foo1` will get foo2's result and return `1*3`.

![Call stack visualisation](https://media.giphy.com/media/5nh6BTAoVp1ewnwtO0/giphy.gif)

## Back to callbacks

#### What should I be passing as an argument now? :cold_sweat:

Say we have these 2 functions:
```js
function firstFunction(){
 console.log("I'm the first function");
}

function secondFunction(callback){
 callback();
 console.log("I'm the second function and expect a callback as an input");
}
```
`firstFunction` is just your normal everyday function, while `secondFunction` gets a callback as an argument.
If you want the second function to run properly, you'll have to give it a function and so calling `secondFunction(firstFunction)` will call the first function logging `I'm the first function` then logs `I'm the second function and expect a callback as an input` as expected :tada:.

Well, that wasn't so hard, let's pick it up a notch with three functions, adding this one:
```js
function thirdFunction(callback) {
 callback();
 console.log("I'm the third function, I'm also expecting a callback as an input");
}
```
So, what happens when calling `thirdFunction(secondFunction(firstFunction));`?

Oops! we got an error, `TypeError: callback is not a function at thirdFunction`, but why?

Well `thirdFunction` expects a callback and we passed `secondFunction(firstFunction)` which evaluates and returns a `void` (`void` is equivalent to an empty `return;`), or in simpler words this isn't a function!

_How to fix it?_
We'll have to send a function instead 
```js
thirdFunction(function(){
 secondFunction(firstFunction);
});
```
`thirdFunction` expects a callback (function) so we pass an anonymous one which then does what we need!
 
#### Don't callbacks get arguments too?
Now that you've mastered callbacks that don't take any arguments, it's time we pass some around :muscle:.
Let's check this example out
```js
function firstFunction(number){
 console.log("I'm the first function and I expect an input, ", number);
}

function secondFunction(callback){
 var number = 1;
 callback(number);
 console.log("I'm the second function and expect a callback as an input");
}

secondFunction(firstFunction);
```
`firstFunction` expects an input that's why we have `callback(number)`, you're familiar with the rest :relieved:.

Would this work though? 
```js 
secondFunction(firstFunction(1));
```
Another Error :confused: `TypeError: callback is not a function at secondFunction`, same error as before, we need a function and not a value.

## Time to asynchronous get

You're ready for the hardest part, it's when things get asynchronous and confusing

#### Step I: Synchronous vs. Asynchronous

We're used to think about code that runs beautifully, executing line after another making it logical and easy to understand. This is synchronous code, it executes the way we read it.

Imagine you're baking a cheese cake, the cake has a base and a cheese mix on top, if you'd like to bake this cake synchronously, you would first prepare your base and bake that, wait until it is finished then prepare the cheese, you add the mix on top and then bake it, and if you want to be extra, you'll wait for that to finish baking and prepare some strawberry syrup to pour on top...
Sounds like lots of things, why not prepare things while the base is baking?

If you're already thinking that, congratulations you understand what asynchronous is, it is when you're able to execute other tasks while one (maybe more) is being processed.

In a nutshell, 
- synchronous code waits for 1 action to complete before moving on to the next
- asynchronous code doesn't have to wait and can execute multiple actions at the same time

#### Step II: A taste of asychronousity

Below we see two logs and we expect the result to be `Hello World` (separated by a new line of course)
```js
console.log("Hello");
console.log("World")'
```

We'll use `setTimeout` to execute the first log in 3 seconds instead of immediately, the code would look like this:
```js
setTimeout(function(){
 console.log("Hello");
}, 3000);

console.log("World");
```

run these lines and you'll get `World` logged before `Hello` because the first log will take 3 seconds to run.

Asynchronous calls work in a similar way, with the difference that we don't know when we'll get the result back.

#### Step III: Dive in

Here's an asynchronous function (it's not really async but simulates the same behaviour)
```js
function asyncAddOne(x, callBack) {
 setTimeout(function() {
   return callBack(x + 1); 
 }, 200)
}
```
What happens here is the callback will execute after 2 seconds, anything synchronous will execute regardless, this bit of code might make it clearer
```js 
var x = 0;
  asyncAddOne(1, function(param){
    x = param;
  });

console.log(x);
```
Wait a second, what just happened? it logged 0! 
let's break it down:
- we're assigning 0 to x
- then calling `asyncAddOne` with 1 as an input 
- the callback assigns the returned value to x
- x doesn't change :grimacing:

Which means that the `log` has already executed while `asyncAddOne` was still waiting in the task queue :no_good:.

__How can it be done then?__

As you might have guessed, it'll be possible to use from within the callback itself...
```js
asyncAddOne(1, function(param){
 console.log(param);  
});
```
it works!

Let's double the fun and add another async function
```js
function asyncMultiplyThree(x, callBack) {
 console.log("result from first function ", x);
 setTimeout(function() {
   return callBack(x * 3);
 }, 200)
}
```
Try to multiply the result from the first function using the second function, the result should be 6.
Again we'll need to pass the result from within the first function's callback, meaning we can only call `asyncMultiplyThree` from `asyncAddOne`'s callback and it looks like this

```js
asyncAddOne(1, function(arg){
  return asyncMultiplyThree(arg, function(result){
    console.log(result);
  })
});
```
## Done

These are all the basics you need to start hacking some callbacks!
Use it wisely and don't worry about callback hell :wink:.
