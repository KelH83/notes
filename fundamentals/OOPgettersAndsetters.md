# Getter & Setters

## Prior Knowledge

-   know ES6 class syntax
-   be able to use inheritance to extend classes from one another
-   understand what encapsulation is and why we strive to use it

## Learning Objectives

-   Understand how we can achieve encapsulation using private properties
-   Able to limit a our interactions with properties using getters and setters

## Background

**Encapsulation** is one of the four fundamental pillars of OOP. In functional programming, and using factory functions we have achieved this by creating **closures**.

### Private Properties

A common practice that has been adopted in JS is to prefix a property or method with an underscore. This tells another developer that this field is required for internal use and shouldn't be read or updated. We can use the same syntax for methods which we deem to be private.

```js
const createCounter = () => {
	const counter = {};
	counter._count = 0; // counter._count, please don't access this directly!
	// ...
	return counter;
};
```

We can achieve _true encapsulation_ by creating private fields which cannot be accessed outside of the class declaration. We do this using **# name** syntax. In order to assign a value to a private field within the constructor method, it must first be declared within the class.

```js
class TimeSheet {
	#hoursWorked = 0;
	#hourlyRate;
	#shiftLength = 8;

	constructor(name, hourlyRate) {
		this.name = name;
		this.#hourlyRate = hourlyRate;
	}
}

const davidsTimeSheet = new TimeSheet("David", 9.9);

davidsTimeSheet.name; // "David" - public field
davidsTimeSheet.hoursWorked; // undefined - private field
```

Defining private fields prevents them from being read or reassigned in the same way we can with public fields. In fact, if we `console.log` the instance, they wouldn't be visible.

### Getters and Setters

We can control how a user can interact with our private fields by creating predefined behaviours which will expose the variables. We could consider building the following method to get the number of hours worked:

```js
// ...
	getTotalHours() {
		return `${this.name} has worked ${this.#hoursWorked} hours.`;
	}
// ...
davidsTimeSheet.getTotalHours()

```

A more common approach to do this is using the accessor function syntax known as a `getter`. A getter allows us to define a function that will be invoked when we access it using it's name.

```js
// ...
get totalHours(){
    return `${this.name} has worked ${this.#hoursWorked} hours.`;
}

// ...

davidsTimeSheet.totalHours // "David has worked 40 hours."
```

Using the getter syntax we are able to invoke it's functionality as if it were actually a property. This is commonly used when the data can be calculated from a private variable.

```js
// ...
get totalPay(){
  return `£${this.#hoursWorked * this.#hourlyRate}`
}

davidsTimeSheet.totalPay // "£396"
```

If we wish a user to be able to set a value we can use the `setter` syntax. Similar to the getter syntax, we can interact with the object as if we were reassign a property, when infact it is invoking the setter function with the assigned value.

```js
// ...
set shiftsWorked(numOfShifts){
    this.#hoursWorked = this.#shiftLength * numOfShifts;
 };
// ...

davidsTimeSheet.shiftsWorked = 4;
```
