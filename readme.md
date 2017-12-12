# Building a MERN App

## Quick References

- [Deploying a Node-Express-Mongoose App/API with Heroku & MLab](https://git.generalassemb.ly/ga-wdi-lessons/express-mongoose-mlab-deploy)

## Learning Objectives (5 min / 2:35)

- Describe the difference between single-server and multi-server configurations
for an application built in the MERN stack
- Install and use `cors` to allow for Cross-Origin Resource Sharing between our
front-end application and back-end API
- Use `npm run build` to compile our front-end application into an optimized and
deployable directory
- Describe the process of unifying our front-end and back-end code into a single
project
- Set up our front-end development server to proxy requests to our back-end server
- Set up our back-end API to serve static `build` assets in production

## Framing (10 min / 2:45)

To integrate React with a back-end framework (such as Express) we will need to make
a few decisions about the desired architecture of our application. The first primary
decision to make is where we want our front-end application to be served from. There
are two primary options:

**Multi-Server (Microservice) Configuration** - our front-end application and back-end
API will be completely separate. They will each have their own Git repositories and will
be deployed independently to different servers. Our front-end will retrieve data from our
back-end across separate domains.

  - **Pros**
    - Separation of Concerns: Our application is more modular and our front-end
    and back-end developers can work on their parts of the application independently
    of each other.
    - Specialized Configuration: Since our front-end and API will be housed on separate
    servers, we can optimize server configuration for each one independently, allowing them
    to scale independently of each other and preventing them from competing over the same
    server resources.
  - **Cons**
    - Since requests will be sent across separate domains, we will have to configure CORS
    for the browser to allow our front-end to retrieve data from our back-end  


**Single-Server (Monolithic) Configuration** - our front-end application will be housed in the same
repository as our back-end code. They will be deployed together to a single server where
our back-end will be responsible for serving up our front-end application in addition to our
API data.

  - **Pros**
    - Single Deployment: Since our front-end will be served up by our API, we only need to
    manage one deployment.
    - Unified Codebase: Since our front-end and back-end will be housed in the same Git repo,
    we will always have a more full picture of the application than if they were in separate repos.
    - No CORS: Since our front-end application will be making requests back to the same
    domain that served it up (its 'origin'), we do not need to configure CORS.
  - **Cons**
    - We will have to configure our development environment to allow us to use the
    features of our Webpack server provided by `react-scripts` (automatic compiling,
    linting, hot reloading, etc.).
    - We will have to modify our server to serve up both json data for the API endpoints
    and the assets of our front-end which could require additional configuration and cause
    the two to compete over server resources.

Today, we will walk through setting up and deploying our application up first on separate servers, then examine how to combine them into one project and deploy onto a single server.

## Getting Started (10 min / 0:55)

Today, we will be using the React Translator app that used the IBM Watson API to translate text and provide audio pronunciations. We will also be using a Mongoose / Express back-end to allow for users to save translations.

### Back-End Setup

1. First, clone down the back-end, install dependencies, and open in atom:

    ```bash
    git clone git@git.generalassemb.ly:ga-wdi-exercises/react-translator-api.git
    cd react-translator-api
    npm install
    atom .
    ```

2. Check to make sure `mongod` is running.

    Type in `mongo` and if it drops you into the
    mongo shell, you're good to go. Otherwise, open a **separate** tab in your terminal
    and run `mongod`. Leave this tab running for the rest of this exercise.

3. Run the seeds file to populate our MongoDB database with translation data.

    ```bash
    node db/seeds.js
    ```

4. Start the server

    ```bash
    npm start
    ```
    > If you inspect `package.json`, you will see that this is an alias for `node index.js`

5. Navigate to `localhost:3001/api/translations` to see the json our back-end is serving


### Front-End Setup

1. In a ***separate terminal window***, clone down the React Translator app, checkout to the `mern-starter` branch, install dependencies, and open in atom:

    ```bash
    git clone git@git.generalassemb.ly:ga-wdi-exercises/react-translator.git react-translator-front-end
    cd react-translator-front-end
    git checkout mern-starter
    npm install
    atom .
    ```

2. Start the development server:

    ```bash
    npm start
    ```
    > If you inspect `package.json`, you will see that this is an alias for `react-scripts start`

3. Navigate to `localhost:3000` to explore the application


## Application Dive (5 min / 3:00)

With a partner, take 5 min to look through our back-end and front-end apps.  Try to answer the following questions.
* What npm packages are we using?
* What are the properties for our model?
* What components are we using in React and how are they nested?
* What CRUD functionality is currently available to us on the back-end?
* Where is the front-end making calls to our back-end?

## Two-Server (Microservice) Architecture (30 min / 3:30)

Currently we are using this type of architecture. Our back-end is running on `localhost:3001`
while our front-end is running on `localhost:3000`. One way to say this is that these applications
have different "origins". One issue with this is that our browser is not going to like requests from
our front-end (served by `localhost:3000`) to our back-end on `localhost:3001`. In the application,
navigate to the Saved Translations view and check the console. You should see:

```
XMLHttpRequest cannot load http://localhost:3001/api/translations. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:3000' is therefore not allowed access.
```

This is Chrome telling us that since our back end is running on a separate port ("origin") than our front end,
our front end is not allowed to retrieve data from it. To fix this, we need to configure CORS (Cross-Origin Resource Sharing) on our back-end.

### Installing `cors` in Express

1. Stop your Express server, and install `cors` via npm:

    ```bash
    npm install --save cors
    ```

2. In `index.js`, require the `cors` module and integrate with Express:

    ```js
    const cors = require('cors')
    ```

    ```js
    app.use(cors())
    ```

3. Restart your Express server:

    ```bash
    npm start
    ```

Now when you navigate to `localhost:3000/translations`, you should see a list of
the translations being retrieved from our API.
> Note: The default `cors` configuration (above) will allow requests from **any** origin (which may or may not be ideal). To more precisely control access to our API, we would need to do little more configuration. Check out the [cors documentation](https://www.npmjs.com/package/cors) for more information on this process.

### Deploying to Separate Domains

Now that our front end and back end are communicating properly, let's look at how we would
deploy them.

#### Back End Deployment

As our back-end will need to be hosted on a server and host a database, we will want
to deploy it to a hosting service like Heroku. We would go about this is no differently
than we have previously:

```bash
heroku create react-translator-api
git push heroku master
```
> Note: Heroku will require some additional configuration to host a MongoDB database.
It will prompt you to set up an account with a cloud hosting service [mLab](https://mlab.com/welcome/?gclid=EAIaIQobChMI5tT3qfiV1gIVTjPTCh3tlAM7EAAYASAAEgIRkfD_BwE).
For details on this, check out the [Heroku Documentation](https://devcenter.heroku.com/articles/mongolab).


#### Front-End Deployment

Currently, when we start our React application locally, we are using `react-scripts`. `react-scripts` is the black box that contains all of the major dependencies and configuration that we are using in development:
- Dependencies: Babel, Webpack, and ESLint and other dependencies that allow the use of JSX, hot reloading, compilation of our application, and other features.
- Configuration: Configuration files for the above dependencies that control (among other things) how our app is compiled in both development and production environment.
- Scripts: Commands that allow us to do things like start our Webpack server (`react-scripts start`) or create a compiled, minified version of our application (in vanilla JS) that can be deployed (`react-scripts build`).

The command that we will need to use for deploying our front-end is `react-scripts build`. `create-react-app` automatically aliases this for us in our `package.json` as `npm run build`. In the terminal, run this command to create a deployment-ready version of your app:

```bash
npm run build
```

This will create a directory called `build` in the root of your application. This directory contains all of your application's HTML, minified JS, and minified CSS in a deployment-ready format.

Now all you would need to do is to deploy **the build directory** to a static asset hosting service (such as [GitHub Pages](https://pages.github.com/) or [surge](https://surge.sh/).

If pushing to gh-pages:

```bash
git subtree push --prefix build origin master:gh-pages
```
> Note that `subtree` allows us to only push a sub-directory of our repository instead
of the entire app. [More Information](https://gist.github.com/cobyism/4730490)

If using surge:
```bash
npm install -g surge
surge
```
> [Surge](https://surge.sh/) is a CLI based npm package that lets you quickly deploy static front-end
applications for free.

## Break (10 min / 3:40)

## Single-Server Architecture (45 min / 4:25)

Now let's look at how we could consolidate our front-end and back-end into one project and deploy them together to one server. First, let's copy our React Translator app into our API repo:

From the root of the `react-translator-api` directory, run:
```bash
cp -r <path to the `react-translator` directory> ./client
```
> Note that we are both copying the front-end repo to our root folder while
simultaneously renaming it to `client`

### Configuring Express to Serve Static Assets

Now, in a deployed production environment, we want our express server to serve up
the static assets in `client/build`. In `index.js`, let's set up a route to do this:

In `index.js`
```js
app.get('/', (req, res) => {
  res.sendFile(__dirname + '/client/build/index.html')
})
```

Now when we navigate to `localhost:3001`, we will see errors in the console saying that
the browser cannot load our minified JS and CSS files. To fix this, we need to tell express
where to look for static assets:

In `index.js`
```js
app.use(express.static('client/build'))
```

Reload the page and we will see that our app is working as intended. However, before we deploy,
we will need to update the paths of any requests in our front-end to be *relative* instead of *absolute* since we are now serving both the app and data from the same origin.

For example, in `client/src/components/Translations/Translations.js`, the url `axios` is querying
should be changed from
```js
axios.get('http://localhost:3001/api/translations')
```
to
```js
axios.get('/api/translations')
```
> Hint: use CMD + SHFT + F to find all instances of a url in a project

In order for this change to be reflected via `localhost:3001`, we would not have to run
`npm build again` since our Express app is serving assets out of the `client/build` directory.
This is cumbersome and not something we should have to do **every time** we make a change
to our front-end code.

To use hot reloading and linting like we're used to, we would
have to change into the `client` directory and run `npm start` to start up our Webpack development
server on `localhost:3000`. This would be fine but it would break all of our relative paths that we
just defined above for requests (the API is at `localhost:3001/api/translations` not `localhost:3000/api/translations`). How do we get around this?

### Proxying Requests

One easy solution is to allow our Webpack server (on `localhost:3000`) to *proxy* requests for
our API server (on `localhost:3001`). What this means is that if a request is made to `localhost:3000`
at a path that it doesn't recognize, it will simply forward that request to `localhost:3001`, and then
relay the response back as well. In essence, our Webpack server will serve as a middleman between the browser and the API **only during development** so that we can use all of the bells and whistles that `create-react-app` provides to us.

In `client/package.json`, add this line:
```json
"proxy": "http://localhost:3001/"
```

Then run `npm start` within client and navigate to `localhost:3000` in your browser. Our data is being successfully retrieved!

We will still need to run `npm run build` inside of `client` whenever we want to update our static production
code in `build`, but for quickly iterating in development, we now can quickly see our updates rendered.

## Bonus: Run through MLab Deployment

[Deploying a Node-Express-Mongoose App/API with Heroku & MLab](https://git.generalassemb.ly/ga-wdi-lessons/express-mongoose-mlab-deploy)

## Bonus: Create Script to Start Up in Development

Since we are only going to need to start up both our Express server and our Webpack server
in development, we might want to set up a script to run both of these process *concurrently*:

Install the `concurrently` package via npm:
```bash
npm install --save-dev concurrently
```

In `package.json`, lets add a command to the `scripts` object:
```json
"scripts": {
  "start": "node index.js",
  "test": "echo \"Error: no test specified\" && exit 1",
  "dev-start": "concurrently \"npm start\" \"cd client && npm start\""
},
```
> Note that `&&` allows us to chain commands in bash

Now, to start up both servers for development, all we have to do is run:
```bash
npm run dev-start
```

## Closing / Questions (15 min / 2:00)

## Resources

**Microservice Solution**
- Back-End: https://git.generalassemb.ly/ga-wdi-exercises/react-translator-api/tree/microservice-solution
- Front-End: https://github.com/ga-wdi-exercises/react-translator/tree/mern-starter

**Single-Server Solution**
- Back-End: https://git.generalassemb.ly/ga-wdi-exercises/react-translator-api/tree/single-server-solution
