# Introduction to CI/CD

## Learning Objectives

- Know what CI/CD is
- Understand the benefits of CI/CD
- Know how to and understand why to hide secrets on the build server

## Continuous Integration, Delivery and Deployment

Continuous integration is the process of merging software changes into the main repository and verifying the correctness of the software through automated tests. Changes should be integrated as often as possible, ideally multiple times a day. The more quickly changes can be integrated, the lower the risk of introducing big problems due to large changes to the main codebase. Tests are usually run on a dedicated server so that the integration environment is common for every engineer working on the application.

Continuous delivery is a practice where the software application can be deployed easily at any time with no manual intervention from engineers. As soon as the tests have passed after integrating a change then it should be possible to deploy that change. The idea is that the faster the change can be in front of the end users of the software then the faster feedback can be gathered and acted upon - one of the central ideas of Agile.

Continuous delivery is often seen as a natural progression from continuous integration and continuous deployment is the next natural progression from continuous delivery. The idea behind continuous deployment is that every change in version control is automatically deployed all the way through to production without any intervention from the engineers.

Both continuous delivery and deployment are frequently referred to as pipelines with different stages. They rely heavily on automated testing at each stage to decide whether the change to the software can continue to the next stage in the pipeline. Running the unit tests would most likely be at the start of the pipeline, then the next stage might be deploying the software. There could be a number of different environments that the application is deployed to before it gets to production and in front of real users.

## The CI/CD environment

To implement a CI/CD pipeline there needs to be a way for checks to run whenever a change is made to a repository. There are many products for this and they exist in a fairly complicated and interactive ecosystem. These include the extremely feature-rich and mature [Jenkins](https://www.jenkins.io/), and the newer kid on the block, [CircleCI](https://circleci.com/); there is also [Code Pipeline](https://aws.amazon.com/codepipeline/) and all of these can be integrated with GitHub. GitLab provides its own CI/CD tools internally, alongside its repository management. To keep things familiar, these notes use [Github Actions](https://github.com/features/actions) - tools that were previously used to manage workflows around GitHub, like issues and project boards, but which have fairly recently been repurposed by GitHub to facilitate CI/CD.

All of these different services have slightly different features, syntaxes and implementations. But there are many considerations that will exist no matter which you choose, and we may not always do things the optimal 'GitHub Action way' so you are exposed to some of these considerations that aren't so neatly abstracted in other pipelines.

## GitHub Actions

To get GitHub to trigger actions on a repository, the repository needs a directory named `.github` at the root level, and inside that, a `workflows` directory. _Workflow_ is the name given to what might be called a _pipeline_ with other providers. There may be multiple pipelines for a project. Your repo may be responsible for multiple applications deployed separately, for example, or you could separate the CI and CD parts of your pipeline. Each workflow needs a [YAML](https://yaml.org/) file to configure it - the steps in this example will be fairly simple, so it can be created in a single workflow.

Let's name it `test-and-deploy.yml`. Each `workflow` will need `name` property:

```yaml
name: Test & Deploy
```

The pipeline can be triggered by various events in the GitHub architecture. Most commonly for CI/CD pipelines, you would use `push` and `pull_request` events.

There are some checks that you may wish to run every time your code is pushed to the main repository, whether that is the `main` branch, or some other branch. This is part of the _continuous integration_ of the code into the repository. These may include unit tests, or using tools for _static code analysis_ - tools that looks at the code without running it - such linting, assessing vulnerabilities, and other code quality checks.

Other processes may only make sense to run once we're happy that the code has passed previous checks, plus any other manual code reviews that need to be undertaken anyway - this might involve a _staging area_ so checks for things like UX and accessibility can be signed off, or it might mean straight into production, as we will do - that's `continuous deployment`!

As this workflow will handle all of the CI/CD, the workflow file can be configured to run whenever a specific event happens. In this case, some code being pushed to the `main` branch:

```yaml
on:
  push:
    branches:
      - main
```

Other events that can trigger actions can be found in the [GitHub Actions documentation](https://help.github.com/en/actions/reference/events-that-trigger-workflows).

### Jobs

GitHub Actions allows you to define separate tasks, or _jobs_ that you might want to undertake in your pipeline. With this structure you can separate the processes from each other, and provide different contexts or environments for them to be undertaken in.

The first job we will specify is running the tests, as we want to make sure no broken changes can be deployed. Under a `jobs` key, we will define a mapping with `<job_id>` keys to reference each job. First:

```yaml
jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
```

On this `test` job property we have defined a human-readable name, used by the GitHub UI to inform us about the pipeline's progress. We've also provided the OS for the job to be run on. We should generally be emulating the environment of our deployment. On GitHub Actions, Linux machines are the cheapest, so `ubuntu-latest` will do fine.

Jobs can be further broken down into _steps_ that need to be undertaken sequentially. The first step for our code is fairly specific to GitHub Actions - we need to _checkout_ the code that we are working with. This may seem redundant, but remember Actions are also for managing projects, issues and other parts of the GitHub system that don't require code to work on.

Luckily, the code for doing this has already been abstracted into a pre-existing Action, and this is one great feature of GitHub Actions - common tasks are likely to have been written, tested and evaluated by the community already, meaning that you can piece together your own workflow from other people's efforts. We'll avoid that in some places today but we will use this Action across our pipeline so the abstraction is very useful.

```yml
lint:
  name: Lint
  runs-on: ubuntu-latest
  steps:
    - name: Checkout Repo
      uses: actions/checkout@v3
```

The syntax for the Action referred to under `uses` is `<owner>/<repo>@<branch>` - anything with an `owner` of `actions` is an official GitHub action.

Other pre written Actions can be found in the [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)

The next step is to set up a node environment. Again, there is another common GitHub-made Action we can use here. We can provide information to the action using the `with` key - here we'll use the latest node version (at the time of writing!). Always check the documentation for pre-built actions, so you know what information needs to be provided, and how.

If the tests make use of a database, the it will also be important to provision a database as part of the `job`. Thankfully, there is also a handy [PostgreSQL GitHub Action](https://github.com/Harmon758/postgresql-action) that can be used. For this, it is necessary to define the name of the database that needs to be created, as well as a user & password to allow access to it:

```yaml
steps:
  - name: Checkout Repo
    uses: actions/checkout@v3

  - name: Use Node.js
    uses: actions/setup-node@v3
    with:
      node-version: 18

  - name: Use PostgreSQL
    uses: harmon758/postgresql-action@v1
    with:
      postgresql db: 'my_db'
      postgresql user: 'test_user'
      postgresql password: 'test_password'
```

Note that the password and username does not need to be hidden here because they are only for use on a test database that will quickly be disposed of.

(It's worth noting that there is a very powerful syntax for defining different runtimes for your code, something that could be particularly useful if you wanted to check a native application on various device OSs, for example. To do this requires defining a _matrix_ of different OSs and runtimes, and GitHub Actions will test on all of these concurrently. You can read more about this [in the docs](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstrategy).)

There are two more steps for the job: install dependencies and run the test script. Make sure you have a `test` script in your `package.json`!

When running the `test` script, environment variables need to be provided that will enable connection to the database created in the `Use PostgreSQL` step.

In the workflow file, we now have a complete job:

```yml
test:
  name: Test
  runs-on: ubuntu-latest
  steps:
    - name: Checkout Repo
      uses: actions/checkout@v3

    - name: Use Node.js
      uses: actions/setup-node@v3

    - name: Use PostgreSQL
      uses: harmon758/postgresql-action@v1
      with:
        postgresql db: 'my_db'
        postgresql user: 'test_user'
        postgresql password: 'test_password'

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: PGDATABASE=my_db PGUSER=test_user PGPASSWORD=test_password npm t
```

`npm ci` is a similar command to `npm install` that is recommended for use in CI/CD environments. It will rely upon a `package-lock.json` and only install the specified versions of listed dependencies. More about the `npm ci` command [here](https://docs.npmjs.com/cli/v9/commands/npm-ci)

## Deployment

If you have hosted your application with Render one of the features it offers is automatic deployments when code is pushed to a branch. This is an example of continuous deployment setup by Render and works very well. 

This feature will deploy the code from our main branch, but we doesn't run any checks on our code. As an alternative we can trigger a manual deployment on render using a [webhook](https://render.com/docs/deploy-hooks).

We can make a request to this endpoint from our CI to deploy our code on render once all of our checks have passed. This way we are sure that everything is working as we expect before deploying.

### Secrets

The endpoint that render gives us is unique to our application and as such should be kept secret. Anyone with this endpoint could re-deploy our app at any point so we absolutely do not want it to be committed to our code or otherwise viewable, such as through the logs that GitHub provides when running its Actions. The solution is to declare them in [_Secrets_](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository), which you can get to from the sidebar under the settings tab for your repo.

To access secrets in our pipeline we use the `${{ secrets.NAME_OF_SECRET }}` syntax. These values will be masked in the logs of the pipeline to keep them safe and can be used as part of jobs. 

For example we can use the `curl` terminal command to make a request to the deploy endpoint.

```yml
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Render
        run: curl $\{{ secrets.RENDER_DEPLOY_HOOK_URL }}
```

### Action sequencing

In a pipeline, each step should be giving us confidence that the code is of sufficient quality to apply the next check. There's no hard and fast order that things should happen, but it makes sense that inexpensive (both in time and money, as the free GitHub Action minutes will not last forever) tasks should complete before moving onto tasks that take longer.

With the example above, there would be nothing preventing both jobs running at the same time but it is usually the case that the quality checks should pass before deploying the changes. To do so, a `needs` property can be added to a `job`. The value of the `needs` property should refer to any `jobs` that must be completed successfully before the workflow undertakes this job:

```yml
  deploy:
    runs-on: ubuntu-latest
    needs: test # wait until test is successful before running this job
    steps:
      - name: Deploy to Render
        run: curl $\{{ secrets.RENDER_DEPLOY_HOOK_URL }}
```