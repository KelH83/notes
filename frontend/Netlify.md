# Deploying Your React App with Netlify

There are numerous ways to deploy a React app. The [vite documentation](https://vitejs.dev/guide/static-deploy.html) has a list of options for deployment. Here, we have chosen to use [Netlify](https://vitejs.dev/guide/static-deploy.html#netlify) because it is convenient and has a free tier.

## 1. Setup Redirects

Add a file, `_redirects` (no file extension) to your `public` directory.
This file should contain the redirect rule:

```
/* /index.html 200
```

This is telling Netlify "if a request comes in to _any_ endpoint on our base url - serve our index.html page and give a 200 status".
We put this in the `public` directory to ensure that Webpack includes this file in the production build of the app.

## 2. Create a Production Version of Your App

```bash
npm run build
```

This script complies your code into a few uglified "static" files that can be read by most modern browsers.
Take a look inside - but don't change anything.

## 3. Create a Netlify Account

[Sign up to Netlify](https://app.netlify.com/signup). You can do so easily with GitHub.

## 4. Install Netfify's CLI

Netlify provide a command line interface for performing common operations on their service, such as deploying your site.

To [get started](https://cli.netlify.com/getting-started) install the CLI with npm:

```bash
npm install netlify-cli -g
```

## 5. Deploy to a Draft URL

```bash
netlify deploy
```

- Authorise Netlify with GitHub, following the prompts in the browser.
- Select `Create & configure a new site`.
- Provide your choice of site name.
- Select your personal account.
- Provide a deploy path. This needs to point to your build directory and should be `./dist` (as we're in the root of our react app).

Your draft version should now be deployed on a url, e.g. `https://5c13ab16055b9be1725868e6--your-site-name.netlify.com`.
Test it out, make sure that everything is working as expected.

## 6. Deploy to a Production URL

`netlify deploy --prod`
Specify your build path again.
This will deploy the site to your actual url: `https://your-site-name.netlify.com`.

## Redeployment

1. Create an updated build version of your code:

```bash
npm run build
```

2. Deploy to a draft url:

```bash
netlify deploy
```

3. Deploy to your production url:

```bash
netlify deploy --prod
```
