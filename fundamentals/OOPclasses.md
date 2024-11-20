## Classes

Classes are [syntactic sugar](https://en.wikipedia.org/wiki/Syntactic_sugar) for JavaScript's prototypal inheritance. The syntax was introduced in ES6 to help those more familiar with other object oriented programming (OOP) languages easily adopt OOP. Like constructor functions, class definitions should be written with a capital letter.

### The `constructor` Method

A `class` definition requires a `constructor`. This is the method that is called when _instantiating_ the class, and behaves the same way any constructor function used outside of a class.

```js
class Mammal {
	constructor(name) {
		this.name = name;
		this.warmBlooded = true;
	}
}
```

### Instantiating Classes

To create a new instance of a class, use the [`new` operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new).

The `new` keyword does the following things:

1. Creates a blank, plain JavaScript object.
2. Links (sets the constructor of) this object to another object.
3. Binds the value of `this` in the constructor to equal the new object.
4. Returns the new object if the function (or class) doesn't return its own object.

A class's constructor can accept parameters, just like a constructor function would:

```js
const mouse = new Mammal();
console.log(mouse); // { name: undefined, warmBlooded: true }

const rat = new Mammal("Squeaker");
console.log(rat); // { name: 'Squeaker', warmBlooded: true }
```

To put functions on the prototype of the class add them (without the `function` keyword) outside of the constructor:

```js
class Mammal {
	constructor(name, favouriteFood) {
		this.name = name;
		this.warmBlooded = true;
		this.favouriteFood = favouriteFood;
	}
	eat() {
		return `${this.name} takes a big bite of ${this.favouriteFood}`;
	}
}

const rat = new Mammal("Squeaker", "cheese");
rat.eat(); // 'Squeaker takes a big bite of cheese'
```

## Principles of OOP

There are four key concepts that dominate how you should construct object oriented systems. These become more important to understand in fully OOP languages like C# and Java, but JavaScript makes some accommodations for them too. The first, **encapsulation**, is something we already understand from the more 'functional programming' perspective - it's good practice to separate behaviour that is unrelated into different methods. `class`es should be treated similarly - only keep related behaviours and properties in one place. This makes finding behaviour more predictable.

The others are **abstraction**, **inheritance** and **polymorphism**. JavaScript doesn't make many allowances for polymorphism in its design (though _overriding_ could be considered this) so it is enough to understand that it involved allowing different objects to share variations of the same behaviour.

### Abstraction

To understand the importance of abstraction, it's important to understand that Javascript has no concept of _private variables_ - in JS, when an object is created, its properties are always accessible.

In OOP, this can be undesirable. When we are creating methods to interact with these properties, it is because we want to define the specific ways users are allowed to mutate these properties. This is all very well, but it doesn't stop a user from directly accessing the property and overwriting it with whatever they want.

There are two ways to encourage people not to do this, but neither are fool-proof. The first is to indicate that a variable should not be accessed or changed directly, by preceding it with an underscore:

```js
class Mammal {
	constructor(name, favouriteFood) {
		this._name = name;
		this._warmBlooded = true;
		this._favouriteFood = favouriteFood;
	}
	eat() {
		return `${this._name} takes a big bite of ${this._favouriteFood}`;
	}
}
```

This is entirely a matter or convention though, and doesn't actually stop a user from doing something like `myMammal._favouriteFood = 'Cheerios'`.

Javascript also has the concept of **accessors** (also known as _getters_ and _setters_). These are special methods encapsulated within the class that intercept read and write operations on the object:

```js
class Mammal {
	constructor(name, favouriteFood) {
		this._name = name;
		this._warmBlooded = true;
		this._favouriteFood = favouriteFood;
	}

	get favouriteFood() {
		return this._favouriteFood;
	}

	set favouriteFood(newFood) {
		this._favouriteFood = newFood;
	}
}

const me = new Mammal("human", "Weetabix");
console.log(me.favouriteFood); // will return Weetabix
```

Note how I am not logging `me._favouriteFood`, which would also return Weetabix, but would be directly accessing a property with an underscore. Note also that I am not invoking `me.favouriteFood`, but that nevertheless attempting to read this property invoked the accessor function.

As these accessors are functions, you can perform any logic in them before reading or writing to the object. You may wish to validate a value before using it in a `set`, for example, or format a value before accessing it in the `get`.

### Inheritance

#### Extending Classes

One of the most useful things about classes is how you can create subclasses. This is particularly helpful if you've chosen an example as clearly taxonomical as animals.

To create a new class, which has all of the methods and properties from another class, use the `extends` keyword.

```js
class Dog extends Mammal {
	bark() {
		console.log(`${this.name} woofs!`);
	}
}

const otis = new Dog("Otis", "doggy treats");
otis.bark(); // 'Otis woofs'
```

In the example above, `Dog` extends `Mammal`. Therefore, when `Dog` is instantiated, the constructor of `Mammal` is called with the same arguments. This means that `Dog` has both name and the favourite food available, and because it is prototypically linked to `Mammal`, it will also have the methods from `Mammal` available on it.

#### Passing Arguments to Parent Classes

However, if the subclass needs to use a constructor of its own, we run into a bit of a problem. For example, we might want to give our dog an owner property, so it can whine for them.

If `Dog` is only called with owner as an argument, it won't know its name or favourite food. We want to call a constructor in our dog, and then pass the information that information that the Mammal constructor needs upwards.

For this, use `super`. `super` invokes the parent constructor, so you should pass `super` the params that the parent class requires. `super` must be invoked before any reference to `this` in the child constructor.

```js
class Dog extends Mammal {
	constructor(name, owner) {
		super(name, "doggy treats");
		this.owner = owner;
	}
	bark() {
		return `${this.name} woofs!`;
	}
	whine() {
		return `${this.name} misses ${this.owner} desperately`;
	}
}

const otis = new Dog("Hamish", "Mitch");
otis.bark(); // 'Hamish woofs!'
otis.whine(); // 'Hamish misses Mitch desperately'
```

### Overriding Methods

Finally, it is possible to "override" the methods on the parent class. Simply add a method of the same name to a sub class.

When a method is invoked, JS will look up the prototype chain until it finds the corresponding method (or reaches the end of the chain).

```js
class BigDog extends Dog {
	bark() {
		return `${this.name} WOOFS!`;
	}
}
const wolf = new BigDog("Ghost", "Jon Snow");
wolf.eat(); // 'Ghost takes a big bite of doggy treats'
wolf.whine(); // 'Ghost misses Jon Snow desperately'
wolf.bark(); // 'Ghost WOOFS!'
```

As `eat` does not exist on `BigDog`, it goes up from `BigDog` to `Dog` and still doesn't find the a matching method. Then, it moves on from `Dog` to `Mammal` and finds the method there!

If `BigDog` didn't have a bark function on its prototype, the exact same thing would've happened with that. However, because it can find `bark` on its prototype, it doesn't need to look up - it just uses the closest method available on the prototype chain.

## Resources

[This article](https://www.freecodecamp.org/news/object-oriented-programming-concepts-21bb035f7260/) is an excellent language-agnostic introduction to OOP concepts.
