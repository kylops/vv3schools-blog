---
title: How to create and publish a Vuejs component on NPM
date: 2018-03-22 14:08:25
tags: npm
---
Here is one way you can create/publish a Vuejs library/component from scratch.

I made this deliberately long to include all the steps and explain most of the instructions.

If you follow every step of this guide, you will be able to publish your own library/component on npm, and then install it using `npm install mylib` then include it inside your project using the `import myLib from 'my-lib'` keyword and so on..

------

Let us first create a folder call it `vuejs-hello-app` and inside it, run:

```
npm init
```

Just hit enter until the interactive question ends and then npm will generate a file named `package.json` that contains the following code.

> (Note: I gave mine a description, changed version from `1.0.0` to `0.1.0` here is the result.)

```
{
  "name": "vuejs-hello-app",
  "version": "0.1.0",
  "description": "vuejs library demo",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

After this, we'll need to install the dependencies for our library.

These dependencies are divided into two types: `dependency` and `devDependency`

`dependency`:
is the library(s) that our own library runs on. Since we are creating a VueJs component, we need vue as our dependency. So, install it using:

```
npm install --save vue
```

`devDependency`:
is the library(s) we need to create/build/transpire on our local machine. We install dev dependencies using the method above but adding the the suffix `-dev` to `--save`

Now, let us install some dev dependencies:

```
npm install --save-dev babel-core
npm install --save-dev babel-loader
npm install --save-dev babel-preset-env
npm install --save-dev cross-env
npm install --save-dev css-loader
npm install --save-dev file-loader
npm install --save-dev node-sass
npm install --save-dev sass-loader
npm install --save-dev vue-loader
npm install --save-dev vue-template-compiler
npm install --save-dev webpack
npm install --save-dev webpack-dev-server
```

At this point our `package.json` will be updated with the information about our library's dependencies and should look like following.

```
{
  "name": "vuejs-hello-app",
  "version": "0.1.0",
  "description": "vuejs library demo",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack -p"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "babel-core": "^6.26.0",
    "babel-loader": "^7.1.2",
    "babel-preset-env": "^1.6.1",
    "cross-env": "^5.1.1",
    "css-loader": "^0.28.7",
    "file-loader": "^1.1.5",
    "node-sass": "^4.7.2",
    "sass-loader": "^6.0.6",
    "vue-loader": "^13.5.0",
    "vue-template-compiler": "^2.5.9",
    "webpack": "^3.10.0",
    "webpack-dev-server": "^2.9.7"
  },
  "dependencies": {
    "vue": "^2.5.9"
  }
}
```

> (note: I have added `"build": "webpack -p"` to build our lib with webpack)

Now, since our code needs to be build and transpiled, we need a folder to store the build version. Go ahead and create a folder inside our root folder and call it: `dist` and in the same place a configuration file for webpack and name it `webpack.config.js`

All of the files we have sofar created are for configuring and stuff. For the actual app that people are going to use, we need to create at least two files inside our `src/` directory.

A `main.js` and `VuejsHelloApp.vue` put them as: `./src/main.js` and `./src/components/VuejsHelloApp.vue`

I have mine structured like this.

```
dist
node_modules
src
  main.js
  components
    VuejsHelloApp.vue
.babelrc
.eslintignore
.gitignore
.npmignore
.travis.yml
CONTRIBUTING
LICENSE
package.json
README.md
webpack.config.js
```

I will just go through the files listed and describe what each file does in-case anyone is curious:

`/dist` is where a build (transpiled), minified, non-ES6 version of your code will be stores

`node_modules` I think we know this already, let's ignore it

`src/` this is root dir of your library.

`.babelrc` is where your babel options are kept, so add this to disable presets on modules

```
{
  "presets": [
    [
      "env",
      {
        "modules": false
      }
    ]
  ]
}
```

`.eslintignore` This is where you tell ESLINT to ignore linting so put this inside:

```
build/*.js 
```

`.gitignore` add files you want to ignore (from git)

`.npmignore` same as .gitignore for NPM

`.travis.yml` if you need CI check [examples](https://docs.travis-ci.com/user/languages/javascript-with-nodejs/) from travis and configure it

`CONTRIBUTING` not required

`LICENSE` not required

`package.json` ignore for now

`README.md` not required

`webpack.config.js` This is the important file that let's you create a build, browser compatible version of your code.

So, according to our app, here is a minimal example of what it should look like:

```
var path = require('path')
var webpack = require('webpack')

module.exports = {
  entry: './src/main.js',

  module: {
    rules: [
      // use babel-loader for js files
      { test: /\.js$/, use: 'babel-loader' },
      // use vue-loader for .vue files
      { test: /\.vue$/, use: 'vue-loader' }
    ]
  },
  // default for pretty much every project
  context: __dirname,
  // specify your entry/main file
  output: {
    // specify your output directory...
    path: path.resolve(__dirname, './dist'),
    // and filename
    filename: 'vuejs-hello-app.js'
  }
}

if (process.env.NODE_ENV === 'production') {
  module.exports.devtool = '#source-map'
  module.exports.plugins = (module.exports.plugins || []).concat([
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: '"production"'
      }
    }),
    new webpack.optimize.UglifyJsPlugin({
      sourceMap: true,
      compress: {
        warnings: false
      }
    }),
    new webpack.LoaderOptionsPlugin({
      minimize: true
    })
  ])
}
```

Note that the important directives here are `entry` and `output`. You can check webpack docs to learn more if you want to fully customize your app.

But basically, we're telling webpack to get the `./src/main.js` (our app) and output it as `./dist/vuejs-hello-app.js`

Now, we are almost finished setting up everything except the actual app.

Go to `/src/components/VuejsHelloApp.vue` and dump this simple app, which will move a button right or left when you hover on it

```
<template>
  <div>
    <button @mouseover='move($event)'> I'm alive </button>
  </div>
</template>

<script>
export default {
  data () {
   return {}
  },

  methods: {
    move (event) {
        let pos = event.target.style.float; 
      if(pos === 'left'){
        event.target.style.float = 'right'
      }else{
        event.target.style.float = 'left'
      }
    }
  }
}

</script>

<style scoped>

</style>
```

And not but not least, got to `./src/main.js` and export your app like:

```
import VuejsHelloApp from './components/VuejsHelloApp.vue'
export default VuejsHelloApp
```

Now go to your `package.json` file replace the `"main: "index.js",` with `"main": "src/main.js",`

After this, simply run these commands to build and publish your app:

```
npm run build
git add .
git commit -m "initial commit"
git push -u origin master
npm login 
npm publish
```

------

### Importing and using the library.

If everything went smoothly, then simply install your app like this:

```
npm install --save vuejs-hello-app
```

And use it in vue like this: