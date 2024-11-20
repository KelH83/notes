## Front-End Project Intro

## Prior Knowledge

- Be able to plan a React application's component tree.
- Be able to use side effects in React components.

## Learning Objectives

- Understand how to enable CORS (Cross Origin Resource sharing) in an express server.
- Be able to use a separate file for functions making API requests.
- Understand how to build reusable React components.

## CORS

You will come across a ['cross-origin resource sharing'](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) (CORS) error message when you try to get data from your back-end API. This is because, by default, `express` blocks any requests made to it from another domain (at this point, probably `http://localhost:3000`).

To allow CORS on your back-end API for NC News/ NC Games:

```bash
$ npm install cors
```

In your `app.js` file, require in the package:

```js
const cors = require('cors');
```

In your `app.js` file, have your app use the `cors` middleware before any of your other middleware or routers.

```js
app.use(cors());
```

Once you have made these changes you will need to commit and push them to take effect in your hosted api.

```sh
$ git add app.js
$ git commit -m 'allow cross origin resource sharing'
$ git push origin main
```

More info found on the [express documentation](https://expressjs.com/en/resources/middleware/cors.html).


## Logged-in user

Having a user be logged-in to your site is a common feature of websites. As the site your are making is designed to demonstrate your abilities and act as a portfolio piece it is reasonable to make some concessions that we would not normally do in the real world.

People coming to your website for the first time will not know what the existing usernames are and asking them to create an account just to view your portfolio may be an unnecessary barrier to entry. Consider having an existing user that can be logged in by default to make the site more user-friendly.

Think about what kind of functionality is tied to the current user,

- Posting a comment: should the user be logged in in order to post?
- Deleting a comment: should the user be able to delete other user's comments?