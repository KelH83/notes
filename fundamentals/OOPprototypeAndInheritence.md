# Prototypes

## Prior Knowledge

-   understanding of how factory functions can be used to create versions of objects
-   understand how `this` can be used to share function behaviour across different objects
-   the rules of `this` - implicit binding

## Learning Objectives

-   understand how JS uses **prototype** objects in OOP
-   understand how **instances** share methods from their **prototype**

## Background

Unlike programming languages like C or Java, which implement a classical OOP structure, JavaScript makes use of functions and their **prototype** to share functionality. This approach is called **pseudo_classical** and provides a slightly different mechanism to achieve the same thing as factory functions - to easily generate objects with shared behaviours.

### Prototypes

The prototype is an object that exists on all instances of an object. It is designed to house all of the shared behaviours (**methods**) which the instance has access to. The benefit of the prototypal object is that we can declare methods once, and they are shared across all instances of the factory functions.
We can declare a collection of methods in an object like so:

```js
const accountPrototype = {
	checkBalance: function () {
		return `£${this.balance.toFixed(2)}`;
	},
	depositMoney: function (amount) {
		this.balance += amount;
	},
};
```

In order to assign this as a prototypal object for our factory functions we will make use of the `Object.create` method.

```js
// ...
const makeAccount = () => {
	//object instantiation with prototype
	const account = Object.create(accountPrototype);
	// object decoration (adding the properties)
	account.balance = 0;
	//return the object
	return account;
};
```

​
As we've seen previously, `this` will be bound to the context of the object from which the method is called:

```js
myAccount.checkBalance(); // £0.00
myAccount.depositMoney(1.99);
myAccount.checkBalance(); // £1.99
```

We can check which prototype an object has a link to with the `Object.getPrototypeOf` method. It is the same object as `accountPrototype`.

```js
Object.getPrototypeOf(myAccount); // { checkBalance: [Function], depositMoney: [Function] }
Object.getPrototypeOf(myAccount) === accountPrototype; // true
```

```js
console.log(myAccount.__proto__ === accountPrototype); // true
```

This is the link that is used in `Object.getPrototypeOf`. As we have seen, private variables are denoted by an \_. `__proto__` has four of them, indicating we **should not use it**. Instead, use `Object.getPrototypeOf()`.
When you look up a key on an object, it first looks at the object itself to see if it has that property. If that property does not exist then it will look on it's prototype, using `Object.getPrototypeOf` or `__proto__`.
