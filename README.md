# Call it Back

## Contents
1. [Functions as 1st Class Objects](#functions-as-1st-class-objects)
2. [Higher Order Functions](#higher-order-functions)
3. [Synchronous Callbacks](#synchronous-callbacks)
4. [Asynchronous Callbacks](#asynchronous-callbacks)
5. [(Optional) Chaining Callbacks](#optional-chaining-callbacks-on-the-way-to-callback-hell-)
6. [More Resources](#still-not-sure-about-all-that-terminology)

This resource tries to better introduce the concept of callbacks to those new to it, it assumes you have a basic understanding of JavaScript and know what functions are.

To understand it better, try the examples out yourself and even change some of the code to get a better sense of what's happening, when and how. You can try pasting some code samples into [repl.it](https://repl.it) or into the browser console.

## Functions as 1st Class Objects
The term "1st Class Object" might not be very clear. All it means is that everything you can do with an object, you can also do with a function. For example, a function:
- has methods (see [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/prototype#Methods))
- has properties (see [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/prototype#Properties))
- can be assigned to a variable:

```js
function foo() {...}
var someFunction = foo;
```
or
```js
var someFunction = function() {...};
```
(_Notice the second definition uses anonymous functions._)
- And a lot more...

<details>
<summary>More Examples</summary>
<p>

What are these lines doing? can we assign a function to an object?

```js
var obj = {
  1: function () {
    console.log("The first property");
  },
  2: "string"
};

obj[1](); // The first property
```

Answer: Yes you can, if you run these lines you'll get "The first property" logged to your console.
Notice that we invoke the function by adding parentheses to `obj[1]`, without them all we'll get is the function itself (lines of code, that mean nothing if not executed!).

And arrays? can a function be a value in an array?

Yes! it can. What is this code doing?

```js
var functionsArray = [
  function(){console.log("I'm the first value in the array");},
  function(){console.log("I'm the second");},
  function(){console.log("Hello I'm third");}
];

functionsArray.forEach(function(fun){
  fun();
});
```
`fun()` will execute all the functions in the array.

</p>
</details>

## Higher Order Functions
A **callback function**, is a function that is passed as an argument to another function.
A **higher-order function** is a function that takes a callback or return another function.

We already know that a [function can do anything any object can do](#functions-as-1st-class-objects), that also means they can be passed around as arguments!

### Pass a function into another and call it
Let's say we have this simple function:
```js
function run (myCallBack) {
  myCallBack();
}
```
`run` expects one argument as an input, this argument needs to be a function, since we call it.

Trying to execute this line would result in an error:
```js
run("a string");
```
You would see `TypeError: myCallBack is not a function`. This is happening because we use the argument `myCallback` as a function.

#### _How do I pass a function as an argument, though_ 🤔?

As mentioned before, functions are objects in every way, so a function definition can be passed as an argument too:
```js
function theCallBack () {
  console.log("Jack, come back");
}
run(theCallBack);
```
Here we can see `theCallBack` passed to `run` as an argument and as expected this will log `Jack, come back`.

Another way you can pass a function is by using anonymous functions:
```js
run(function(){
  console.log("expected a function, got an anonymous one :|");
});
```
<details>
<summary>More explanation</summary>
<p>

A function doesn't have to have a name! Consider the following example:

```js
function print (x) {
  console.log('Value is' + x);
}
var n = 1;

print(n); // Value is 1
print(1); // Value is 1
```
The function works the same, whether you pass the value directly, or store it in a variable first. Now what if `n` was a function?

```js
function run (x) {
  x();
}

function callback () {
  console.log('Hi');
}

run(callback); // Hi
run(function () { console.log('Hi'); }); // Hi
```
</p>
</details>

Lastly, what is the difference between these two examples? Run the code to find out.
```js
function theCallBack () {
  console.log("Jack, come back");
}

// Example 1
run(theCallBack);

// Example 2
run(theCallBack());
```

### Pass a function into another and call it with arguments
In the [original definition of `run`](#pass-a-function-into-another-and-call-it), we simply executed the callback that was being passed in. But we don't have to do this. We can also pass our own arguments to the callback!

```js
function run (myCallBack) {
  myCallBack('Done!');
}

function logTheArgument (string) {
  console.log('Callback is being run!');
  console.log(string);
}

run(logTheArgument);
```

## Synchronous Callbacks
Synchronous vs Asynchronous code is a big topic and something many people are confused by. That's OK. The following two sections try to introduce the difference, but it will take many people much longer to really understand it. Take your time, practise and experiment.

Synchronous code runs in the order it appears on the page; top to bottom. One command has to finish before the next one can run. This is how most people think about code. All the examples so far in this workshop have been of synchronous code.

Synchronous callbacks are callbacks that are executed by the function they're passed to immediately:
```js
function logArgument (x) {
  console.log('Passed: ' + x);
}

[1, 2, 3].forEach(logArgument);

console.log('Done');
```
Outputs:
```
Passed: 1
Passed: 2
Passed: 3
Done
```
The important thing about this is that because later code has to wait for your callback to finish, you can use `return` to pass data out of your callback and into other code!
```js
function addOne (x) {
  return x + 1
}

function applyTo (callback, value) {
  return callback(value)
}

var result = applyTo(addOne, 2);

console.log('Result is: ' + result); // Result is: 3
```

## Asynchronous Callbacks
With asynchronous code, some code can run before other code has "finished". There'll be a more detailed explanation of what this means later on in the course, but for now, what it means for asynchronous callbacks is:

**The callback is executed AFTER the code below it**

An example, using good old [`setTimeout`](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout):
```js
setTimeout(function () {
  console.log('Timeout over');
}, 1000);

console.log('Finished?');
```
Output:
```
Finished?
Timeout over
```
`setTimeout` is a function that accepts a callback as an argument, but runs that callback _asynchronously_. It waits for the specified timeout (in our case 1000 milliseconds, or 1 second), then runs the callback. This is why `Timeout over` appears after `Finished?` in the output.

What's important to understand here is that, unlike synchronous callbacks, asynchronous callbacks _CANNOT_ use `return` to pass results to other code! Instead, we must pass results to _other_ functions -- _other callbacks_ 🤯

To illustrate this, look at this example. We have an asynchronous function which finishes 1 second in the future, and a callback that we pass into it.
```js
// This is our asynchronous function
function waitOneSecondThen (callback) {
  var result = 10;
  setTimeout(function () {
    callback(result);
  }, 1000);
}

// This is our callback!
function printResult (value) {
  console.log('Result is ' + value);
}

waitOneSecondThen(printResult);
console.log('Starting!');
```
Outputs:
```
Starting!
Result is 10
```
Remember, the _only_ place in our code where we can access `result` is _inside_ `printResult`. Any operation that requires `result` must occur or be called from inside `printResult`. Using `return` doesn't work:

```js
function printResult (value) {
  console.log('Result is ' + value);
  return value;
}

var result = waitOneSecondThen(printResult);

console.log('Starting');
console.log('Return value is ' + result);
```
Outputs
```
Starting
Return value is undefined
Result is 10
```

## (Optional) Chaining Callbacks (on the way to Callback Hell 😈)
There are many patterns that have arisen out the needing to use both synchronous and asynchronous callbacks in Javascript. You'll encounter many as you go through the course, but below is just a very short introduction to one of them.

We've seen [above](#asynchronous-callbacks) how to use a callback to ensure code is run only _AFTER_ an asynchronous function is finished. But it's very common to want to perform two or more asynchronous things one after another. How can we do that?

Well there are several ways, but the one we cover here is to manually chain the callbacks. To begin with, here's an example using _synchronous code_:
```js
function first (callback) {
  console.log("I'm the first function and expect a callback as an input");
  callback();
}

function second (callback) {
  console.log("I'm the second function and expect a callback as an input");
  callback();
}

function third() {
  console.log("I'm the third function, I'm just a regular function");
}
```
What code should we now write so that these functions are executed in order (i.e. `first`, then `second`, then `third`)?

Let's break it down. Look at only `second` and `third`. We can see `second` prints a log, and then runs it's callback. We know that we can use `third` as a callback. So:
```js
second(third);
```
Is the solution we need! Now lets look at `first`. This is very similar to `second`; it prints a log, then runs it's callback. Maybe some of you are thinking we should do this:
```js
first(second(third));
```
But this doesn't work! Why?

Thinking more carefully, we know that whatever function we pass as a callback to `first` needs to run _both_ `second` and `third`. We know that `second(third)` runs both `second` and `third` but it is _not_ a function (check `typeof second(third)` to make sure!). The solution is to wrap this in another function:
```js
first(function () {
  second(third);
});
```
🙌

<details>
<summary>More details</summary>
<p>

Some of you might have noticed that there's another solution:
```js
first(function () {});
second(function () {});
third();
```
This works fine for our current synchronous versions of `first`, `second` and `third`, but wouldn't work if they were asynchronous:
```js
function first (callback) {
  setTimeout(function () {
    console.log("I'm the first function and expect a callback as an input");
    callback();
  }, 10);
}

function second (callback) {
  setTimeout(function () {
    console.log("I'm the second function and expect a callback as an input");
    callback();
  }, 10)
}

function third() {
  console.log("I'm the third function, I'm just a regular function");
}
```
Try it yourself. Can you explain why only one of the solutions works for the asynchronous version?
</p>
</details>


## Still not sure about all that terminology?
Some resources that might help:
- Sync and Async:
  - [Asynchronous JavaScript #1 - What is Asynchronous JavaScript?](https://www.youtube.com/watch?v=YxWMxJONp7E)
  - [asynchronous vs. synchronous execution what does it really mean?](https://stackoverflow.com/questions/748175/asynchronous-vs-synchronous-execution-what-does-it-really-mean)
- Callbacks: [Asynchronous JavaScript #3 - Callback Functions](https://www.youtube.com/watch?v=QRq2zMHlBz4)
