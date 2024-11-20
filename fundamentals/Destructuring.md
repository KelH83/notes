# Destructuring

## Learning Objectives

- understand and be able to use object destructuring
- understand and be able to use array destructuring

## What is destructuring?

> The destructuring assignment syntax is a JavaScript expression that makes it possible to unpack values from arrays, or properties from objects, into distinct variables. MDN

The following are examples of the destructuring assignment syntax used for both objects and arrays.

```js
const tutor = { firstName: 'Paul' };
const { firstName } = tutor;
console.log(firstName); // 'Paul'

const tutors = ['Paul', 'Haz'];
const [tutor1, tutor2] = tutors;
console.log(tutor1); // 'Paul'
console.log(tutor2); // 'Haz'
```

## Object destructuring

Let's say we have a `tutor` object which has keys of `name` and `age`. If we want to keep referring to Shaq's name multiple times, we can assign it this value to a variable like so:

```js
const tutor = { name: 'Shaq', age: 25 };

const name = tutor.name;
console.log(name); // Shaq
```

Another way we can create this `name` variable is through object destructuring. [Object destucturing MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#object_destructuring)

```js
const tutor = { name: 'Shaq', age: 25 };

// no object destructuring
const name = tutor.name;

// object destructuring in shorthand syntax
const { name } = tutor;
```

Here, we're creating a _new_ variable: `name`

But not only are we using object destructuring, we're also using shorthand syntax! The longhand for this object destructuring is:

```js
// object destructuring in longhand syntax
const { name: name } = tutor;

console.log(name); // Shaq
```

In all of these examples above, we're creating a new variable called name, which is equal to the _value_ that's at the _name_ key in the _tutor_ object. (If you use Chrome or Firefox (and perhaps other browsers too), pasting these examples into the 'console' will show any new variables in a different colour which is a useful indicator)

It just so happens that, because we want to take something off the name key AND name that new variable 'name', we can do that shorthand.

```js
const { name } = tutor;
```

### Renaming

This longhand syntax lets us do some fun things! If we're making a brand new variable when destructuring, we can actually use destructuring to rename variables as well.

```js
const { name: firstName } = tutor;
```

Here, we've made a new variable called `firstName`, which holds the value of whatever is at the `name` key in the `tutor` object.

### Default values

If we attempt to destructure a property that doesn't exist on an object, then that new variable will be assigned the value `undefined`.

```js
const tutor = { name: "russ" };

const { name, pets } = tutor;

console.log(name); // 'russ'
console.log(pets); // undefined
```

Here, we've created two new variables: `name` which holds the value of the `name` property of `tutor` and `pets` which is `undefined` because there is no `pets` property on `tutor`.

When destructuring, we can assign a _default_ value which will be used if the unpacked value is `undefined`. If the value is defined, then the default value will be ignored.

```js
const tutor = { name: "russ" };
const { name = "default name", pets = [] } = tutor;

console.log(name); // 'russ'
console.log(pets); // []
```

This is similar to the example above, but because `pets` is not defined on `tutor` it will be assigned the default value of `[]`.

### Destructuring multiple things

The biggest benefit of destructuring is that we take multiple values off objects in one go:

```js
const { name: firstName, age: yearsLived } = tutor;
console.log(firstName);
console.log(yearsLived);
```

If we didn't care about renaming `name` and `age` to something different, we can do that shorthand:

```js
const { name, age } = tutor;
```

### Nested destructuring

If we want to extract a value that's deeply nested inside an object, we can use destructuring to get that value as well.

```js
const tutor = { name: 'Shaq', age: 25, petCount: { lizard: 3, cat: 1 } };

// If I want access to just the number of lizards Shaq has
const {
  petCount: { lizard: lizardCount },
} = tutor;
```

This makes me a new variable called `lizardCount`, which is equal to the `value` at the key of `lizard`. This `lizard` key is inside an object which exists at the `petCount` key ... which is inside the tutor object!

You can probably tell from that description that this is getting very nested - if you're destructuring from a very deeply nested object, then you might risk losing readability!

Note that in the above example, we have _only_ made one new variable: `lizardCount`. We haven't made a new variable for the petCount variable. Trying to `console.log(petCount)` here would lead to a reference error, because this variable still only exists as a key inside the tutor object.

### Within function parameters

Destructuring can be very useful within functions.

```js
const tutor = { name: 'Shaq', age: 25, petCount: { lizard: 3, cat: 1 } };

const logName = (personObject) => {
  const { name } = personObject;
  console.log(name);
};

logName(tutor);
```

If we _only_ care about the value at that `name` key, and no other values inside the personObject, then we can actually destructure within the parameters themselves.

```js
const tutor = { name: 'Shaq', age: 25 };

const logName = ({ name }) => {
  console.log(name);
};

logName(tutor);
```

This means that _straight away_ we get the variable that we want which is nice, but note that it does mean that don't have access to the `age` property anymore.

## Array destructuring

We can also destructure from arrays - [Array Destructuring MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#array_destructuring)

```js
const tutors = [
  { name: 'Shaq', age: 25 },
  { name: 'Izzi', age: 23 },
];

// without array destructuring
const shaqObj = tutors[0];

// with array destructuring
const [shaqObj] = tutors;
```

By putting the [] on the left side, this makes me a new variable `shaqObj` , holding the value in the first position of the array.

We can also take multiple things off the array;

```js
const [shaqObj, izziObj] = tutors;
```

## Super advanced array destructuring

If we only wanted the `izziObj` variable and not the `shaqObj` then we could simply not write a variable name for the first position and just have a comma:

```js
const [, izziObj] = tutors;
console.log(izziObj); // { name: "Izzi", age: 23 }
```

An alternative way of destructuring makes use of the fact that all arrays are _objects_ and so the indexes of an array are essentially to the keys on that array object. If we had a very long array and wanted to access a variable at a specific index (e.g. 'manchester' which is at the 5th index of the `cities` array), we could do this:

```js
const cities = [
  'cardiff',
  'london',
  'edinburgh',
  'bristol',
  'liverpool',
  'manchester',
];

const { 5: bestCity } = cities;
```

This makes us a new variable `bestCity` which is equal to the value at the key `5` in the cities 'object'. Because `cities` an _array_, the key `5` means the 5th index, meaning we can directly access that 'manchester' string.

## Nested array destructuring

If we have a nested array and we only want to access a variable a few levels deep, we can do nested destructuring with arrays too:

```js
const arrayOfArrays = [
  [1, 2, 3],
  ['a', 'b', 'c'],
];

const [[one]] = arrayOfArrays;
console.log(one); // 1
```

Both object and array destructuring let you have neater code, and are particularly powerful if you're taking lots of values off an object or off an array!
