# SOLID principles of OOP

​

### Introducing SOLID

​
Some of the most well-known and followed principles with regard to how we structure our OOP code are from Robert C. Martin (Uncle Bob) and are commonly known as the SOLID principles.
​

This is an acronym for
​

- Single Responsibility
- Open/Closed
- Liskov Substitution
- Interface Segregation
- Dependency Inversion
  ​

We do not have in JS the concept of interfaces so we will not be looking at them today, but many other languages do.
​

## S - Single Responsibility Principle

​
"A class should have one and only one reason to change, meaning that a class should only have one job".

​
This is like our small, single purpose functions. These are functions that should do one job and do it well! In this same vein of thinking, our "classes" should have a single purpose.

​
Imagine in our sprint we did not create a Pokeball class, the logic in our trainer may look something like this.
​

```js
​
class Trainer (){
    constructor(name){
        this.name = name
        this.pokebelt = []
    }
    catch(pokemon){
        if(this.pokebelt.length === 6){
            console.log("no empty pokeballs")
        }else{
            this.pokebelt.push(pokemon)
            console.log(`you caught ${pokemon.name}`)
        }
    }
    getPokemon(name){
        const pokemon = this.pokebelt.find((pokemon)=>pokemon.name === name)
        if(pokemon === null){
            console.log(`you dont have a pokemon called ${name}`)
        }else {
            return pokemon
        }
    }
}
```

​
That seems reasonable but what if now we decide that we need to add an element of chance to how we catch a pokemon, maybe you only catch the pokemon 6/8 of the times, or if we decide that trainers now need to be able to keep track of the battles they have had ...
​

Here we seem to be changing our class for two very different purposes. Changing it to add functionality for the information that a Trainer has or functionality of a Trainer and if we have changes that are about the mechanism of catching pokemon ... Thus we created the Pokeball class.
​

Some helpful tips for figuring out when you know you might be doing too much in a function or a class:
​

- If you try and describe the function or class and you need to use the word 'and'.
- For every abstraction you create, if there is a useful part which can be extracted in to an even smaller function/class.
  ​

That doesn't mean that a single class cannot encapsulate several things.
​

If you had a user that had a favourite TV, Movie and Song, you might be tempted to split these up into three different classes: Video, Audio, TV.
​

But instead you could create a Media class instead. It is all about finding a balance.
​

## O - Open-closed Principle

​
"Objects or entities should be open for extension, but closed for modification."

​
Open for extension means that we can extend functionality (i.e. able to add new features) without breaking the code base.
Closed for modification means that we should only make non-breaking changes to existing code.
​

Overall, it means we design and write code that lets us extend without having to modify what's already there.
​

Imagine we live in a world with only 2 types of Pokemon, Fire and Water. Sorry Bulbasaur.
​

```js
class Battle {
  constructor(trainer1, trainer2, trainer1PokemonName, trainer2PokemonName) {
    this.trainer1 = trainer1;
    this.trainer2 = trainer2;
    this.trainer1PokemonName = trainer1PokemonName;
    this.trainer2PokemonName = trainer2PokemonName;
  }
​
  fight() {
    const pokemon1 = this.trainer1.getPokemon(this.trainer1PokemonName);
    const pokemon2 = this.trainer2.getPokemon(this.trainer2PokemonName);
    if (pokemon1.type === "water" && pokemon2.type === "fire") {
      // is super effective damage
    } else if (pokemon1.type === "grass" && pokemon2.type === "fire") {
      // not very effective damage
    } else {
      // is normal damage
    }
  }
}
```

​
This is a simple but naive battle class. Now imagine Bulbasaur is finally ready to get involved, we now could extend our program to handle grass types. The way we have designed our code goes against the open for extension, closed for modification as we are going to need to add further if's to our fight method.
​

As a simple fix, we could have instead designed our pokemon to have isEffectiveAgainst and isWeakTo methods.
​

```js
class Battle {
  fight(attacker, defender) {
    let extra = 0;
    if (pokemon1.isEffectiveAgainst(pokemon2)) {
      extra = attacker.attackDamage * 0.25;
    }
    if (pokemon1.isWeakTo(pokemon2)) {
      extra = attacker.attackDamage * -0.25;
    }
    return defender.takeDamage(attacker.attackDamage + extra);
  }
}
```

​
A good rule of thumb can be that if you, as a user, need to open up a JavaScript file and change the code in it so that it can be extended, then you are breaking this principle.
​

## Liskov substitution principle

​
"Let q(x) be a property provable about objects of x of type T. Then q(y) should be provable for objects y of type S where S is a subtype of T".

​
In JavaScript (and other languages), we can 'shadow' a "parent's" method with its own (called overridding). The Liskov principle says that when you do this, you should do it in a way that doesn't break functionality from a client's point of view.

​
Another way to understand this, is:

​
A program should be able to replace any instance of a parent class with an instance of a child without negative side-effects.
If a Fiat500 is a subtype of a Car, then anywhere a car is used, it should be able to be replaced with a Fiat500.
​

We currently can use all the Pokemon subclasses in the same way, they all have the same methods, this is intentional, we should be able to use a Bulbasaur in the same way we interact with any given Pokemon. This could become an issue if we created a Pokemon that was able to use multiple moves ...
​

```js
class Trubbish extends Pokemon {
  constructor(name) {
    super(name);
    this.attackDamage = 9001;
    this.hitPoints = 1000;
    this.move1 = 'tackle';
    this.move2 = 'be awesome';
  }
  useMove(moveNumber) {
    console.log(`${this.name} used ${this[`move${moveNumber}`]}`);
  }
}
```

​
Whilst this might not be a fantastic implementation we can see that normally we would invoke any Pokemon's useMove method with nothing, but this Class needs us to pass it an argument. Therefore we cannot use a Trubbish in place of any Pokemon and would result in breaking our program.
​

Lets look at how we might be able to make Trubbish fit with this principle:
​

```js
class Trubbish extends Pokemon {
  constructor(name) {
    super(name);
    this.attackDamage = 9001;
    this.hitPoints = 1000;
    this.move1 = 'tackle';
    this.move2 = 'be awesome';
  }
  useMove(moveNumber = 1) {
    console.log(`${this.name} used ${this[`move${moveNumber}`]}`);
  }
}
```

​
Another example taken from [here](https://softwareengineering.stackexchange.com/questions/170222/what-can-go-wrong-if-the-liskov-substitution-principle-is-violated):
​

## Dependency Inversion principle

​
Entities must depend on abstractions not concretions and high-level modules should not depend on low-level modules. Both should depend on abstractions.
​

This basically means that we design our functions to concepts instead of a specific implementation.
​

Our Trainers specification said that they should have 6 pokeballs by default, an implementation of this could look like the below
​

```js
class Trainer {
  constructor(name) {
    this.name = name;
    this.pokebelt = [
      new Pokeball(),
      new Pokeball(),
      new Pokeball(),
      new Pokeball(),
      new Pokeball(),
      new Pokeball(),
    ];
  }
}
```

​
In this case our Trainer class is very much reliant on bringing in the Pokeball constructor. If we wanted to add the ability for trainers to start with different types of Pokeball or make different types in the future we would have to go back and change our Trainers implementation.
​

What we could do instead is pass in the Pokeball constructor or an array of Pokeballs to the trainer class. This would mean that at the point of creating the trainer we can pass in those things in at the point of us using the Trainer class. Inverting the dependency of Trainer and Pokeball
​

## Further reading

​

- [solid](https://medium.com/@cramirez92/s-o-l-i-d-the-first-5-priciples-of-object-oriented-design-with-javascript-790f6ac9b9fa)
- [Liskov](https://www.youtube.com/watch?v=Mmy1EUKC_iE)
- [TDA](https://martinfowler.com/bliki/TellDontAsk.html)
