# Scope

## Prior Knowledge

- Understand the difference between a function declaration and a function expression

## Learning Objectives

- Understand certain features and behaviours of JS like hoisting
- Understand what we mean by scope in javascript
- Understand how JavaScript engine looks up variables in code
- Understand what is meant by block scope

## Scope


We often describe variables as being **in scope** or **out of scope**.  **Scope** refers to the variables that are accessible to us at some point in our code.  When we refer to the **scope of a variable** we mean the region of the program in which we can directly access the variable.

For example:

```js
function sayName() {
  const name = 'Mitch';
  console.log(name); // 'Mitch' - the variable 'name' is in scope at this point.
}

sayName();
```

We may already intuitively know that the `'name'` variable is **in scope**, simply because it was declared and assigned a value on the line above.

### Local Scope

If we try to log `name` outside of this function then the javascript engine will throw a `ReferenceError`. This indicates that a variable cannot be found in our code.

```js
function sayName() {
  const name = 'Mitch';
}

sayName();

console.log(name); // ReferenceError - the variable 'name' is no longer "in-scope".
```

In the above example, the function `sayName` has its own **local scope**. Local scope refers to the variables that are directly accessible within the function body of `sayName`. When `sayName` is invoked and we enter its local execution context, the variable of `name` is created but once we have closed the execution context, `name` is not accessible in the global scope. Thus, the variable `name` is said to be contained within the local scope of `sayName`.

### Parent Scope

In contrast, if instead we declare `name` outside of `sayName` then it does not have `name` in its local scope.  However, despite the fact that the variable is not declared in the local scope of `sayName` we still get `"Mitch"` printed to the console.

```js
const name = 'Mitch';

function sayName() {
  console.log(name); // logs 'Mitch'!!
}

sayName();
```

This works because the variable `name` is still in scope, at the point the thread reads the `console.log`.

If there is no `name` variable available in the **local scope** of the function, then the JS engine will attempt to find it elsewhere. The next place it is going to look is the outer scope or the **parent scope** of `sayName`. The **parent scope** in this example happens to be the global scope.  Once the variable `name` is found in the parent scope then the javascript engine will stop looking for this variable

All variables that have been declared in the global scope are in scope at any point in our code.

### Shadowing

Another situation that can occur is if there is a variable with the same name as a variable in an outer scope.

```js
const name = 'Mitch';

function sayName() {
  const name = 'Tom';
  console.log(name); // logs "Tom"
}

sayName();
```

This behaviour is known as shadowing. Since a variable called `name` is available in the local scope of `sayName`, there is no need to go searching elsewhere to find it.

However, the following example shows how even though we haven't passed an argument to `sayName`, a variable of `name` is still declared inside of `sayName` (the function's argument), but assigned `undefined` by default when `sayName` is invoked with no arguments:

```js
const name = 'Mitch';

function sayName(name) {
  console.log(name); // logs "Undefined"
}

sayName();
```

When the following code is read by the JavaScript engine, the `changeName` function has to find the variable `name` from outside of its local scope before it can reassign it. The variable still exists in the global scope, and `name` is reassigned from `'Izzi'` to `'Mitch'`.

```js
let name = 'Alex';

name = 'Izzi';

const changeName = function() {
  name = 'Mitch';
};

changeName();

console.log(name); // logs 'Mitch'
```

### Lexical Scoping

JavaScript follows a **lexical scoping** model for scope. Lexical scopes means that the scope of variables is determined at **author time** when functions are declared.  The variables that are accessible is a direct result of how we choose to nest function declarations.

Consider the example below: 

```js
let num = 10;

const printNum = function() {
  console.log(num);
}

const func = function () {
  let num = 3;
  printNum();
}

func(); // console.logs --> 10
```
We can break down the above example in the following way:
  1. `num` is declared and initialised with a value of `10`
  2. `printNum` and `func` are declared, pointing to functions
  3. `func` is invoked in the global, opening a local execution context
  4. Inside, `func` a local variable called `num` is declared and initialised with a value of `3`
  5. `printNum` is then invoked, opening a local execution context for `printNum`
  6. `num` is then logged

In order to be absolutely certain which value of `num` will be logged we need to understand precisely how lexical scoping works.  The scope of `printNum` was created at **author time**, where the function is declared.  If we examine where `printNum` is declared in our code we see that it is nested inside the global variable environment.  This means that `printNum` can only access its local variables and any variables defined in its parent scope, which is just the global scope in this example.  Therefore `num` gets logged as `10`.  The fact that `num` is declared as `3` inside `func` is irrelevant: scope is determined by where our functions are declared and which other scopes they were nested inside.

However, now consider the following modified example below:

```js
let num = 10;

const func = function () {
  let num = 3;
    const printNum = function() {
      console.log(num);
    }
  printNum();
};

func(); 
```

There is subtle difference with the above code and how it was written at **author time** that means there we will see variables being looked up in a different way.

  1. `num` is declared and initialised to `10`
  2. `func` is declared and points to a function
  3. `func` is invoked and a local execution context is created
  4. Inside `func`'s local execution context a variable called `num` is declared and initialised to `3`
  5. `printNum` is declared and points to a function
  6. `printNum` is invoked and then `num` is logged

`printNum` will have access to the variables in its parent scope which we can find by looking at where `printNum` was declared.  In this particular example, `printNum` was declared nested inside `func` and therefore `printNum` will be able to look out of its local scope and find the variable `num` declared inside `func`'s local execution context.

In alternative models of scope like **dynamic scoping** the value of `num` would be `3`.

### Block Scopes

Since the introduction of `let` and `const` declaration keywords in ES6, we can now create block scopes. This means that any variable declared within a block, such as a for-loop or a while-loop, is only available within the scope of that block. For example...

```js
function countToTen() {
  for (let i = 1; i <= 10; i++) {
    console.log(i); // 1.. 2.. 3.. 4.. etc..
  }
  console.log(i); // ReferenceError!!
}

countToTen();
```

Above, the `i` variable has been declared using `let`, and therefore can only be accessed within the `for` loop

Attempting to log `i` outside of the `for` loop gives a `ReferenceError` since it was declared with `let` and therefore can only be referenced within the block it was defined.

Altering the declaration of `i` to a `var` declaration confirms that block-scoping only occurs when we declare with `let` and `const`

```js
function countToTen() {
  for (var i = 0; i <= 10; i++) {}
  console.log(i); // 11 !!
}

countToTen();
```

`var` declarations are **NOT** block scoped. However they _still_ conform to the other rules of scope. The variable `i` in this case is only available within the local scope of `countToTen`.

### Non formal declarations (variable leaking)

If we do none of the above and declare a variable without a formal keyword `var`, `let`, or `const` then the variable we meant to declare gets assigned to the global scope.

```js
function func() {
  if (true) {
    name = 'mitch';
  }
}
func();
console.log(name); // "mitch"
```

This outcome is highly undesirable as we now have a variable exposed to the global where it could be mutated.  It also makes no sense for this variable to exist in the global scope if it is only meant to exist for use within `func`'s local execution context.  Using the `let` and `const` keyword ensures that  

## Hoisting

**Hoisting** is a term that refers to a specific behaviour in javascript whereby variable declarations are brought to the top of the context in which they are declared. `var` and **function declarations** are made accessible at the very beginning of the code in which they were declared. This behaviour can be demonstrated with the following examples:

### Function Declarations

```js
console.log(sayName); //"[Function sayName]"

function sayName() {}
```

The function `sayName` is **hoisted** to the top of the code, allowing it to be accessed before its written declaration.

```js
sayName(); //logs 'Name!'

function sayName() {
  console.log('Name!');
}
```

The above example shows that it's _not_ only the variable `sayName` that is _hoisted_ but also the reference to the function itself.

### Function Expressions

If we use a function _expression_ the function is no longer _hoisted_. E.g.

```js
console.log(sayName); // ReferenceError: sayName is not defined!!
const sayName = function() {};
```

This is because we are using the `const` keyword to declare the function. `const` and `let` declarations are still hoisted but an important feature is they prevent a variable from being used before it is declared by throwing an error.

### Variable declarations using `var`

Variable declarations using `var` also exhibit **hoisting** behaviour: however, there is a difference with function declarations. It is only the variable `a` itself that is hoisted and _not_ the value assigned to it.

```js
console.log(a); // undefined ( NO ReferenceError!)

var a = 100;

console.log(a); // 100
```

We can see this since it prints `undefined` rather than throwing a `ReferenceError`. Remember that `undefined` is often a javascript placeholder for a variable when it hasn't yet been assigned a value.

It is simpler to think of this behaviour as splitting the above expression `var a = 100` into its component declaration (i.e. `var a;`) and assignment parts (i.e. `a = 100`).
