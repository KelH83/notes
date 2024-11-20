# Value vs Reference

## Prior Knowledge

- Identify the 5 main primitive data types in javascript
- How can we check if two quantites are the same in javascript
- What operator can we use to check the type of a particular variable

## Learning Objectives

- Understand the difference between value versus reference
- Understand what it means for a data type to be **immutable**
- Understand how non-primtive data types are held on reference
- Understand the different ways an object can be mutated - dot notation and square bracket notation
- Understand the difference between assertions using `toBe` and `toEqual`
- Understand how to approach testing concerned with immutability

## Data types

The latest ECMAScript standard defines seven data types in the javascript language. These values can be partitioned into two main categories: primitives and objects.

### Primitives

There are 6 primitive data types in javascript:

- String - used to represent a list of characters
- Boolean - this can be either `true` or `false`
- Null - this can only be `null`, used to represent a value of nothing or empty
- Undefined - this can only be `undefined` and means a value has not been assigned
- Number - all numbers fall into this data type, including non-integers and negative numbers
- Symbol (new in ECMAScript 6)

Primitive data types in javascript can be regarded as the fundamental data types. Other more complex data structures will be built up from the primitive data types

#### JavaScript Allows Variables to be Assigned New Values

We can **re-assign** a variable to hold a new value of **any** given data type at any point.

```js
let container = 'fruit';

console.log(typeof container); // "string"

container = 2;

console.log(typeof container); // "number"

container = true;

console.log(typeof container); // "boolean"
```

#### Primitives are immutable

Properties on primitive values cannot be created or overwritten.

```js
const fruit = 'banana';
fruit.age = 28; // write - ignored
console.log(fruit.age); // read - undefined

const name = 'sam';
name[0] = 'S'; // write - ignored
console.log(name); // read - 'sam'

const str = '123';
console.log(str.length); // 3
str.length = 100;
console.log(str.length); // 3
```

#### Primitives are compared by value

To compare primitives, javascript looks at the values of the 2 variables. We can compare two values in javascript using the comparison operator `===` in the following way:

```js
'hello' === 'hello'; // true
'hi' === 'hello'; // false
true === false; // false
undefined === null; // false
```

When variables are assigned to other **primitive** values then the value of the variable is copied over into the other variable.

```js
let a = 10;
const b = a;
a = a * 2;
console.log(a); // 20
console.log(b); // 10
```

## Objects

The remaining data type in javascript is the object. Other data types like **functions** and **arrays** are just special types of objects. Objects can be viewed as a collection of properties that are key-value pairs. The most standard way to create a new object in javascript is by declaring a new object literal using `{` and `}`.

```js
const jonnyLocker = {};
// jonnyLocker is a new empty object
```

### Objects are mutable

One of the most fundamental things about objects is that they are **mutable**. Unlike primitive values we can perform operations on objects that will update their state or content.

#### Dot notation

```js
const jonnyLocker = {};
jonnyLocker.pen = 'biro'; // write
console.log(jonnyLocker.pen); // read - 'biro'
```

In the above section of code, we are mutating the `jonnyLocker` by adding in a key value pair. The use of dot notation means we are adding a key of `pen` to the object and a property value of `'biro'`

#### Dynamic object access

Instead of using dot notation to access or mutate property values in an object we could use square bracket notation. The advantage of using square brackets is that we can use variables to access propery values dynamically:

```js
const jonnyLocker = {};
const itemName = 'pen';
jonnyLocker[itemName] = 'parker fountain pen';
console.log(jonnyLocker); // { pen: "parker fountain pen" }

jonnyLocker[itemName] = 'bog standardard bic';
console.log(jonnyLocker); // { pen: "bog standardard bic" }
```

An **object's keys are always strings** so we could give the `jonnyLocker` object a key using the value of the variable `itemName` which is `'pen'`. We are using square brackets in order to create this key using the value of the variable `itemName`. We can also use square bracket notation to mutate the object by updating its property value to `'bog standard bic'`.

### Objects are held on reference

In javascript, variables that are assigned to primitive data types are assigned the whole value. Any other variable that is assigned the value of the same primitive has its value copied over into the new variable.

Instead, with objects, variables are assigned a **reference** to an object, which is an address or location somewhere in memory. Every time a new object is declared then a new reference is being created that points to a specific address in memory. Consider the following example:

```js
const sam = { name: 'Sam' };
const samDoppelganger = sam;
sam.age = 30;
console.log(samDoppelganger === sam); // true
console.log(sam); // { name : 'Sam', age: 30 }
console.log(samDoppelganger); // { name : 'Sam', age: 30 }
```

On the line with `const sam = { name : 'Sam' };` we declare a new variable `sam` which is assigned a reference to the object `{ name: 'Sam' }`. We could also say that the variable sam points to the object `{name : 'Sam'}`. On the next line the variable `samDoppelganger` is assigned a reference to the `sam` object. In other words, the variable `samDoppelganger` is pointing to the same object as `sam`.

Therefore in mutating the `sam` object so it has an age property with a value of `30`, the console log for `samDoppelganger` will show the same as they are both pointing to the same object in memory.

```js
const paul = { name: 'Paul' };
const paulTwin = { name: 'Paul' };
console.log(paul === paulTwin); // false
```

In the above example, `paul` and `paulTwin` are declared and each time assigned a reference to a new object. Every time a new object literal is declared a new address in memory is being created for that object and so the above objects will not share the same reference.

## Arrays

Arrays are just ordered lists of values. They are also a special type of object and therefore **mutable** in the same way.

```js
const samsFilingCabinet = [];
const mitchsFilingCabinet = [];
console.log(samsFilingCabinet === mitchsFilingCabinet); // false
```

```js
mitchsFilingCabinet.push('p45');
console.log(mitchsFilingCabinet); // ['p45'];
console.log(samsFilingCabinet); // [];
```

We can update `mitchFilingCabinet` by adding `'p45'` to the end of the array: however, in updating `mitchFilingCabinet` we are not updating `samFilingCabinet` as this variable is pointing to an another array in memory.

```js
const samsFilingCabinet = ['books'];
const mitchsFilingCabinet = samsFilingCabinet;
console.log(samsFilingCabinet === mitchsFilingCabinet); // true
console.log(mitchsFilingCabinet); // ['books']
console.log(samsFilingCabinet); // ['books']
```

In the above example, `mitchFilingCabinet` points to the same array as `samsFilingCabinet` and therefore updates to either or these variables are affecting the same array.

## Copying objects

It is often necessary to create copies of objects, another object with exactly the same key-value pairs but with a different reference from the original object.

There are numerous ways of achieving this in javascript, including:

- using a `for` loop to build an array with the same values
- `Array.prototype.slice()`
- `map`/ `filter`/ `reduce` (not recommended - the array creation is usually incidental to the rest of the functionality)
- the spread operator

### Using the spread operator

The spread operator is denoted with 3 dots `...` and is perhaps the most syntatically appealing way of creating a copy of an existing array. In the example below we are building a new array literal and we are using the spread operator to copy the values from `people` into the new array `tutors`.

```js
const people = [
  'Sam',
  'Mauro',
  'Harriet',
  'Mitch',
  { name: 'Jonny', isVerbose: true },
];
const tutors = [...people];
console.log(people === tutors); // false
```

It is important to note that the spread operator only creates a shallow copy of the `people` array meaning that any objects nested inside the array will be copied over by reference.

```js
console.log(people[4] === tutors[4]); // true
```

We can also use the spread operator to copy an object. We can create a new object literal and copy the key-value pairs from the original object into the new object.

```js
const haz = { name: 'Haz', age: 30, hobbies: ['theatre'] };
const hazClone = { ...haz };
console.log(haz === hazClone); // false
```

## Testing and values vs ref

### toBe vs toEqual

In the jest library there are lots of different assertions we can make about different values. Consider the following assertion:

```js
expect([1, 2, 3]).toBe([1, 2, 3]);
```

In the example above we will get an `AssertionError` from jest: this is because `toBe` in jest is checking the references of the two arrays in the same way as the comparison operator `===` does. Each array literal is a new address in memory and therefore jest will throw an `AssertionError` as they are not pointing to each other.

Suppose however we are not interested in the references of the two arrays, instead we are only interested in whether or not their contents are the same. In this case, we can use another assertion in jest:

```js
expect([1, 2, 3]).toEqual([1, 2, 3]);
```

`toEqual` really means **to deeply equal** here and it is checking that the **contents** of both arrays are the same (including order). Deeply means that the arrays will be checked at every level of nesting - i.e. the nested contents will also be checked to see if they are same as the other array. There is no native function that will deeply check the contents of two objects so we have to rely on 3rd party libraries our own implementation of this functionality in order to use it in our code.

#### Returning a new input

Consider a function `createNewPeople` that needs to create a new array of objects but with each age property replaced with a `yearOfBirth` property

```js
const people = [
  {
    name: 'Sam',
    age: 29,
  },
  {
    name: 'Jonny',
    age: 31,
  },
  {
    name: 'Mitch',
    age: 27,
  },
];
```

There are several test cases that are important when implementing a function such as `createNewPeople`. One of the test cases might be in order to check that our function `createNewPeople` returns a **new** array i.e. the output array is pointing to a different array from the input. This will enable us to create a transformed array without mutating the old array.

```js
test('returns a new array', () => {
  const input = ['a', 'b', 'c'];
  const returnValue = createNewPeople(input);
  expect(input).not.toBe(returnValue);
});
```

Here we are checking to see that the return value of `createNewPeople` is pointing to a new array from `input`. If we do not return a new array then when we try and create a transformed array later we would end up mutating the old array.

#### Checking for immutability

Even if we check to return a new array it is still possible we could mutate the input array whilst returing a new array. It is therefore necessary for us to test directly that the input array has not been mutated. A test case could be as follows:

```js
test('does not mutate the input array', () => {
  const input = ['a', 'b', 'c'];
  createNewPeople(input);
  expect(input).toEqual(['a', 'b', 'c']);
});
```

Notice that in this test we are not interested in the return value of `createNewPeople` instead we are concerned with checking that the input array's contents have not been modified by the call to the function. We thus checking that our function has had no side effects of mutating the original input array.
