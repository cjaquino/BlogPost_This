# What is `this` in JavaScript?
This blog posts assumes a basic understanding of JavaScript and seeks to clarify the intricacies of the `this` keyword and alleviate some confusion around the topic.
## A starting point
To start, let's give `this` a definition.
> `this` references the ***object*** that is executing the current function, also known as the ***execution context***.
### Method invocation
Let's explore the definition by creating an object `obj` that has a method called `logger` and invoke it.
```javascript
const obj = {
  logger: function() {
    console.log(this);
  }
}

obj.logger();
```
Invoking `logger` logs `this` to the console. Therefore, we see `obj` logged in the console because it is the object that is executing `logger`. That seems simple enough, but what happens if we invoke a function without a calling object?

### Function invocation
This time, we'll create a standalone function `logThis` that logs `this` to the console.
```javascript
function logThis() {
  console.log(this);
}

logThis();
```
Without a calling object, `this` references the implicit execution context which is the global object. Hence, the global object will get logged to the console. What the global object is depends on where JavaScript is being run. In the browser, the global object is the `window` object while in Node, it is `global`.
## Execution context
The basic examples provided above seem like `this` isn't too complicated. If there's a calling object, then the value of `this` is the calling object; otherwise, it's the global object. However, the execution context isn't always as straightforward. For example, the examples above are both called with their implicit execution contexts. Let's look at **explicit** and **implicit** execution contexts before delving deeper into the more complicated examples.
### Explicit function execution context
Explicit function execution context is when you supply the context for the function to execute in. For example:
```javascript
let obj1 = {
  name: 'Carl',
}

function logThis() {
  console.log(this);
}

logThis.apply(obj1);
logThis.call(obj1);
```
Calling the `apply` or `call` methods on `logThis` explicitly sets the execution context to `obj1` and invoke the function. Without either, invoking `logThis` would log the global object instead. Here's another example:

```javascript
let carl = {
  name: 'Carl',
  logThis() {
    console.log(this);
  }
}

let victor = {
  name: 'Victor',
}

carl.logThis.apply(victor);
```
This time, we're taking the `logThis` method from `carl`, explicitly setting the execution context to `victor`, and invoking it. This code results in the object `victor` being logged to the console. Without `apply`, the invocation of `carl.logThis` would log the `carl` object instead.

### Implicit function execution context
Implicit function execution context means that you are not setting an explicit context and letting JavaScript determine the context implicitly.

As we have seen in the first two examples, the value of `this` is the calling object if there is one or the global object if there isn't (technically, the global object is the calling object in the latter case). Furthermore, in explicit function execution context, we're able to determine the value of `this` by looking at the argument passed in `call` or `apply`.

However, there are times when the execution context can change during runtime and as a result make it difficult to determine what the value of `this` is when a function/method is called implicitly.
```javascript
const carl = {
  logThis() {
    console.log(this);
  }
}

let foo = carl.logThis;
foo();
```
At first it would seem that when `foo` is invoked, it would just invoke `carl.logThis` and log the object `carl` to the console. However, by assigning `carl.logThis` to `foo`, it loses its context and therefore this code will log the global object instead. Let's break the code down a little what happens here
```javascript
const carl = {
  logThis() {
    console.log(this);
  }
}

let foo = carl.logThis; 
// logThis is taken from the carl object and reassigned into foo, losing its connection to carl
// it's similar to just assigning foo a function with the same functionality as carl
// let foo = function() {
//   console.log(this);
// }

foo();
```
We can give it context by explicitly setting it using `call` or `apply` or we can use `bind` (see next section).

Another place context can be lost is from internal function/method calls. 
```javascript
const carl = {
  name: 'Carl',
  logProp() {
    function logName() {
      console.log(this.name);
    }

    logName();
  }
}

carl.logProp();
```
Although, it may seem that `carl.logProp` will log `'Carl'` to the console, it does not and logs whatever the name property of the global object is. This is because `logName` is implicitly invoked without a calling object despite `logProp` executing in the `carl` context.
### Execution context with `bind`
`bind` returns a new function and allows us to hard bind a context to a function. Even when invoked explicitly, the bound function will execute in the context that was passed in `bind`.
```javascript
const carl = {
  name: 'Carl',
  logThis() {
    console.log(this);
  }
}

carl.logThis(); // called implicitly and logs the carl object (calling object).

const victor = {
  name: 'Victor',
}

carl.logThis.apply(victor); //called explicitly and logs the victor object.

const logVictor = carl.logThis.bind(victor);
logVictor();  // logs the victor object because it's the bound context
logVictor.apply(carl); // will still log the victor object despite setting an explicit context
```
## Interacting front-end
We will be using the following HTML.
```html
<!DOCTYPE  html>
<html  lang="en">
<head>
	<meta  charset="UTF-8">
	<meta  http-equiv="X-UA-Compatible"  content="IE=edge">
	<meta  name="viewport"  content="width=device-width, initial-scale=1.0">
	<title>Document</title>
	<style>
		body {
			width: 100vw;
			height: 100vh;
		}
	</style>
</head>
<body>
</body>
</html>
```
Here we have a bookshelf class that is able to add a book named `book`.
```javascript
class  Bookshelf {
  constructor() {
    this.books = [];
  }
  
  addBook(book) {
    this.books.push(book);
  }
}
```
Now let's add some interaction
```javascript
class  Bookshelf {
  constructor() {
	this.books = [];
  }

  addBook(book) {
	this.books.push(book);
  }

# highlight
  addBookOnClick() {
	document.body.addEventListener('click', this.clickHandler);
  }

  clickHandler() {
	this.addBook('book');
  }
# endhighlight
}

# highlight
document.addEventListener('DOMContentLoaded', () => {
  const  b = new  Bookshelf();
  b.addBookOnClick();
});
# endhighlight
```
When the DOM is loaded, we add a click event listener that executes `clickHandler` when the body is clicked. It should then add `book` to the array of books in the `Bookshelf` instance `b`. However, when we click `body`, an error occurs stating that '`this.addBook` is not a function'.

This is another example of context loss that deals with internal functions. On line 21, `addBookOnClick` is executing with `b`  as the context. However, we pass in a function `this.clickHandler` to the click event listener on `body`.  When `body` is clicked, it no longer has context to `b` and therefore calls `clickHandler` with the global object as its execution context. This is why we get the error '`this.addBook` is not a function' because the global object doesn't have an `addBook` function.

To fix this issue, we can bind the `clickHandler` being passed in line 11 to `this` since it has the value of its calling object at this point.
```javascript
addBookOnClick() {
	document.body.addEventListener('click', this.clickHandler.bind(this));
  }
```
Now when we click `body`, it calls the `clickHandler` function that will always execute in the context of `b`.
