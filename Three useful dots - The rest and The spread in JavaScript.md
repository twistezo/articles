# Three useful dots - the rest and the spread in JavaScript

ECMAScript 2015 brought us a lot of news, which resulted in a large number of improvements. Today we will take a closer look at two features that make life easier. Meet the rest paremeters and the spread syntax.

## Rest parameters

The rest syntax allows us to represent an indefinite number of arguments as an array. Take a look to a function that adds all the arguments it is passed.

```js
const sum = (...args) => args.reduce((prev, current) => prev + current);

console.log(sum(1, 2)); // 3
console.log(sum(1, 2, 3)); // 6
```

## Spread syntax

The spread operator allows us to expand iterable objects into individual elements. This functionality is opposite to what we achived with the rest parameters. It can be apply to all iterables such as arrays, objects, sets, maps etc.

```js
const sum = (x, y, z) => x + y + z;
const numbers = [1, 2, 3];

console.log(sum(...numbers)); // 6
```

## Three dots in real use cases

### Copying an array

The spread syntax effectively goes one level deep while copying an array. One level means that the first level of references are copied.

```js
const array0 = [1, [2, 3]];
const array1 = [...array0];

console.log(array1); // [1, [2, 3]]
```

### Creating an array of unique elements

Create the Set which stores only unique elements and convert its back to array.

```js
const array = [1, 1, 2, 3];
const uniqueElements = [...new Set(array)];

console.log(uniqueElements); // [1, 2, 3]
```

### Concatenate arrays

```js
const array0 = [1, 2];
const array1 = [3, 4];
const concated = [...array0, ...array1];

console.log(concated); // [1, 2, 3, 4]
```

### Slicing an array

```js
const [firstElement, ...newArray] = [1, 2, 3, 4];

console.log(firstElement); // 1
console.log(newArray); // [2, 3, 4]
```

We can also remove `n` first elements with comma.

```js
const [, , , ...newArray] = [1, 2, 3, 4];

console.log(newArray); // [4]
```

### Inserting an array at the beginning of another array

```js
const array0 = [4, 5, 6];
const array1 = [1, 2, 3];
const newArray = [...array1, ...array0];

console.log(newArray); // [ 1, 2, 3, 4, 5, 6 ]
```

### Generating an array of numbers from 0 to `n`

```js
const array = [...Array(10)].map((_, i) => i);

console.log(array); // [ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 ]
```

### Destructuring an object

```js
const { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };

console.log(x); // 1
console.log(y); // 2
console.log(z); // { a: 3, b: 4 }
```

### Creating a copy of an object

```js
let person = {
  name: 'John',
  age: 25,
  wallet: {
    sum: 500,
    currency: 'USD'
  }
};
let personCopy = { ...person };

console.log(personCopy);
// {
//   name: 'John',
//   age: 25,
//   wallet: {
//     sum: 500,
//     currency: 'USD'
//   }
// }
```

Note that the copy of the object that is created is a new object with all the original object’s properties but none of its prototypal information.

```js
person.age = 20;

console.log(person.age); // 20
console.log(personCopy.age); // 25
```

### Conditionally adding properties to objects

We can conditionally add properties to a new object that we are creating by making use of the spread operator along with short circuit evaluation.

```js
const colors = {
  red: '#ff0000',
  green: '#00ff00',
  blue: '#0000ff'
};
const black = {
  black: '#000000'
};

let extraBlack = false;
let conditionalMerge = {
  ...colors,
  ...(extraBlack ? black : {})
};

console.log(conditionalMerge);
// {
//   red: '#ff0000',
//   green: '#00ff00',
//   blue: '#0000ff'
// }

extraBlack = true;
conditionalMerge = {
  ...colors,
  ...(extraBlack ? black : {})
};

console.log(conditionalMerge);
// {
//   red: '#ff0000',
//   green: '#00ff00',
//   blue: '#0000ff'
//   black: '#000000'
// }
```

### Splitting a string to characters

This is similar to calling the split method with an empty string as the parameter.

```js
const split = [...'qwerty'];
console.log(split); // [ 'q', 'w', 'e', 'r', 't', 'y' ]
```
