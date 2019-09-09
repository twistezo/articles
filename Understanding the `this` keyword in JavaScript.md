# Understanding the `this` keyword in JavaScript

In most object-oriented programming languages, the keyword `this` has a special meaning. Usually it refers to the object being the execution context (i.e. to the current instance of the object). For example, we use this when referring to an object property from within: we type `this.propertyName`, and then the context is the object and `this` refers to it.

## `this` in JavaScript

In JavaScript it is more complicated because where `this` refers to depends not only on how the function is defined but also on the form of calling it.

Take a look how `this` works depending on invocation place and form.

### Global context

Used in a global context bound to the global object, like `window` in a web browser.

```js
this; // window
```

### Inside object method

Used inside of an object method bound to the closest enclosing object. The object is the new context of the this keyword. Note that you should not change `function ()` to ES6 syntax `fun: () => this.context` because it will change the context.

```js
const obj = {
  context: 'object',
  fun: function() {
    return this.context;
  }
};

obj.fun(); // object
```

In this example with a nested object, `this` still refers to its closest context.

```js
const nestedObj = {
  context: 'parent',
  child: {
    context: 'child',
    fun: function() {
      return this.context;
    }
  }
};

nestedObj.child.fun(); // child
```

### Context-less function

Used inside a function that has no context (has no object as parent) bound to the global context, even if the function is definded inside the object.

Note that we use `var context` instead of `let/const context` because `let` and `const` change the variable enclosed context. `var` is always closest to the global execution context. `let` and `const` declare variables only in a local block scope.

```js
var context = 'global';

const obj = {
  context: 'object',
  funA: function() {
    function funB() {
      const context = 'function';
      return this.context;
    }
    return funB(); // invoked without context
  }
};

obj.funA(); // global
```

### Inside constructor function

Used inside a function that is the constructor of the new object bound to it.

```js
var context = 'global';

function Obj() {
  this.context = 'Obj context';
}

const obj = new Obj();
obj.context; // Obj context
```

### Inside function defined on prototype chain

Used inside a function defined on the prototype chain to creating an object bound to it.

```js
const ProtoObj = {
  fun: function() {
    return this.name;
  }
};

const obj = Object.create(ProtoObj);
obj.name = 'foo';
obj.fun(); // foo
```

### Inside call() and apply() functions

`call()` and `apply()` are JavaScript functions. With these, an object can use methods belonging to another object. `call()` takes arguments separately where `apply()` takes them as an array.

this is here bound to new context changed in `call()` and `apply()` methods.

```js
const objA = {
  context: 'objA',
  fun: function() {
    console.log(this.context, arguments);
  }
};

const objB = {
  context: 'objB'
};

objA.fun(1, 2); // objA, [1, 2]
objA.fun.call(objB, 1, 2, 3); // objB, [1, 2, 3]
objA.fun.apply(objB, [1, 2, 3, 4]); // objB, [1, 2, 3, 4]
```

### Inside bind() function

`bind()` is also a JavaScript method. It creates a new function that will have `this` set to the first parameter passed to `bind()`.

```js
const objA = {
  context: 'objA',
  fun: function() {
    console.log(this.context, arguments);
  }
};

const objB = {
  context: 'objB'
};

const fun = objA.fun.bind(objB, 1, 2);
fun(3, 4); // objB, [1, 2, 3, 4]
```

### Inside event handlers

Used in any event handler (for example, `addeventListener`, `onclick`, `attachEvent`), it is bound to the DOM element the event was attached to.

```js
document.querySelector('.foo').addEventListener('click', function() {
  this; // refers to the `foo` div element
});
```

### ES6 arrow function

Used inside the arrow function it is always bound to its lexical scope. In the arrow function you can’t re-assign the `this` in any way.

```js
const globalArrowFunction = () => this;

globalArrowFunction(); // Window

const obj = {
  context: 'object',
  funA: () => this,
  funB: function() {
    return () => {
      return this.context;
    };
  }
};

obj.funA(); // Window
obj.funB()(); // object
```
