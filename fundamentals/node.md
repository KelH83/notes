# Node

## Prior Knowledge

- Just a vague awareness of how you've run your code in Pre-Course (`npm test`)

## Learning Objectives

- Understand a little of what Node offers in how JS is run.
- Some differences between the Node REPL and running files/modules with node.
- Some uses of npm, a package manager for node.

## What is NodeJS?

NodeJS is a runtime environment for JavaScript. Essentially it's a program that allows our computer to run programs in JavaScript through the Command Line Interface (a.k.a. CLI or terminal).

Up until some years ago, the only environment in which we could run JavaScript code was a web browser, or 'client-side' code. Meanwhile, 'server-side' code - code that dealt with going to a database and inserting or retreiving information - would be coded in a different language such as Java, Ruby or C#.

However, in 2009 someone called Ryan Dahl created **NodeJS**. He was frustrated by some of the limitations of the most popular web server at the time, the Apache HTTP server, and thought that by taking the JavaScript engine from the Chrome web browser and adding a few more specific features, he could create a better environment for executing code on the server.

NodeJS allows JavaScript to be run directly on the computer - not necessarily through a browser.

NodeJS is built on top of Google's V8 JavaScript engine, which is the same engine that the Chrome web browser uses and is generally considered the best optimised (different browsers have different implementations of a JavaScript engine).

With the rise of NodeJS's popularity as an environment for running code, JavaScript started to move towards the server-side of the web too and now we can build whole web applications, front and back end, in JavaScript by running our front-end code in the browser and back-end code with Node.

Node has augmented JS with several features including interacting with databases, networking, and reading and writing files on the computer.

### Running JavaScript code with NodeJS

With NodeJS installed, you can type the `node` command in your Terminal to open the NodeJS **REPL**.

**REPL** stands for Read-Evaluate-Print-Loop. It's essentially a program that waits for the user's input (a line of code), reads it, evaluates it and prints the result. It's a loop because after evaluating each line it will wait for another one. It's also available on browser consoles.

```javascript
> 3 + 3
6    // <-- The code has been read, evaluated and the result has been printed
> 21 * 2
42
>    // <-- This is the REPL waiting for a command
```

- Another way of running JavaScript code is to save a program into a file with a `.js` extension and running it like so:

```
$ node path/to/file/my-program.js
```

This command will evaluate the code in the file, run it and stop.

```javascript
// main.js
3 + 3;
```

However, unlike the REPL, running a JS file with Node won't print everything the JS interpreter reads (the number 6 gets evaluated, but it disappears from memory immediately). We would have to explicitly ask to see it with `console.log`:

```javascript
// main.js
console.log(3 + 3);
```

## The `global` object

When an instance of NodeJS starts running, the `global` object is established as a place where everything you have access to in your programme lives. If you define a variable `myVar`, it will live as a property of the `global` object, so we can also access it as `global.myVar`.

In Node, the `global` object comes with "preloaded" functions and variables like `console`, `process`, `__dirname`, `module` or `require`.

### Modules, exporting and `require`

A particularly useful functionality NodeJS gives us is the ability to bring in code from other files. This is called _importing_ and _exporting_. Seperating code of different concerns into different files is very good practice - it clarifies the purpose of the code, can help make it reusable, and of course, easier to read and find what you're after.

To export from a file, it's simplest to consistently use `module.exports`:

```js
const me = {
  foreName: 'Tyrion',
  lastName: 'Lannister',
};

module.exports = me;
```

You can export any _one_ value from a file. If you want to make more than one thing available, you can include them all in an object.

```js
const me = {
  foreName: 'Tyrion',
  lastName: 'Lannister',
};

const you = {
  foreName: 'Tywin',
  lastName: 'Lannister',
};

module.exports = { me, you }; // this object uses the name of the variable as the key to access it too. It's equivalent to:
/*
module.exports = { 
  me: me,
  you: you
};
*/
```

`module.exports` begins as an empty object - in the above examples we are reassigning it, but we could also attach keys to that object as we could any other:

```js
module.exports.me = {
  foreName: 'Tyrion',
  lastName: 'Lannister',
};

module.exports.you = {
  foreName: 'Tywin',
  lastName: 'Lannister',
};
```

You can use `exports` in place of `module.exports` - `exports` is initially the same reference to `module.exports`, so it's generally a bad idea to use both.

## NPM

NPM (which weirdly doesn't stand for Node Package Manager) is a registry of open source JavaScript libraries that we refer to as "packages".

It's the place where we can see, and more importantly download and use in our own projects, code that other people have written.

One of the unwritten rules of programming is "do not reinvent the wheel" and that's the problem NPM tries to solve: if we need a function to encrypt data and we don't know how to make our own, we can quickly search for someone else's and add it as a dependency of our project.

NPM allows us to install packages at two different levels: global (available to our entire computer) and local (only available to the current project). It's generally a good idea to just install locally - global installs require you to remember to install updates, which is a potential security risk.

### Initialising NPM

Initialise NPM in a project: `npm init`

This command generates a file called `package.json` that keeps tracks of all the dependencies and other meta-data of our project.

Because our dependencies can get quite large, when we share a project over version control we only share our `package.json` file, which has all the information NPM needs to re-download and re-install all the dependencies.

When we install packages with npm (`npm install`), they will appear listed under `dependencies` in the package.json. If what you are installing is a package which will only you as developers will be using (for example, testing frameworks that consumers won't encounter) then it's good practice to tag it as a 'developer dependency' when you install it: `npm install jest -D`.

### NPM Scripts

In the `package.json` file there is another section called scripts. It comes initialised with a `test` script to promote good practice, but you can add any script as an alias for typing code directly into the teminal. For example:

```json
  "scripts" : {
    "start": "node index.js"
  }
```

`npm start` and `npm test` are special cases for script names, because they are so common. You can add any other scripts you like, but to use them, you will need to type `npm run *script-name*`.

Using scripts means you don't need to remember the intricacies of your file structure, and makes it easier to share code and for people to know what to do by pure convention.
