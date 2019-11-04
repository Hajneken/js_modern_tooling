# Notes for modern tooling

## NPM

initialize project

`npm init`

> walks us through questions and creates package.json

### Init npm without questions

`npm init -y`

> best tu run when updating `package.json` with new info

## Git

adding specified file `package.json` to git

`git add package.json`

## Webpack

installing webpack
`npm install --save-dev webpack webpack-cli`

### Starting webpack

> webpack looks for `index.js` and outputs minified version to `./dist/main.js` by default

- we can run webpack from node_modules `node_modules/.bin/webpack`
- better way is to configure script in `"script"` within `package.json`

by default, npm can recognize package needs to be run from `node_modules`

therefore following will suffice

```JSON
"scripts": {
    "build": "webpack"
}
```

we then execute this script by `npm run build`

### Running webpack in development mode

> in production mode (default) webpack minifies our file which is not readable

to build it for dev purposes run

`npm run build -- --mode development`

## Gitignore

> in `.gitignore` specify everything that we don't want to push to remote server

> `.gitignore` is just going to take plaintext paths

**FUN FACT**

> `/.bin` stands for binary which historically refers to executable files
> (despite files in this direcotry need not to be binary nowadays)

### Configuring webpack

> using the file `webpack.config.js` we can configure webpack

We need to be explicit in configuration file

therefore we cannot specify `path:'/dist'` but we need to specify absolute path

like the following

```js
// path module
const path = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    //   __dirname is global variable in node that gives us our current directory
    path: path.join(__dirname, 'dist'),
    filename: 'app.bundle.js'
  }
}
```

**important note:** webpack uses node process and therefore uses `require()` to import modules

> The code intended to run in a browser uses ES module syntax and is transformed with Babel into code that will work in a browser environment

## Babel

> adds ability to compile new JS to older JS

### Installing Babel

install by `npm i -D @babel/core @babel/cli @babel/preset-env`

### Running Babel

run by `node_modules/.bin/babel` or shortcut `$(npm bin)/babel`

to run certain file `$(npm bin)/babel ./src/greet.js`

to run babel in order to return backwards compatible JS

`$(npm bin)/babel ./src/greet.js --presets=@babel/preset-env`

### Configuring Babel to work together with Webpack

by installing `npm i -D babel-loader`

now we have the ability to configure loaders in our `webpack.config.js` to pass our code through before it ultimately gets bundled for distribution

in `webpack.config.js` we add the following

```JS
{
    ...
    module:{
      rules:[
          {
              test: /\.js$/,
              loader: 'babel-loader',
              exclude: /node_modules/,
              options: {
                  presets: ['@babel/preset-env']
              }
          }
      ]
  }
}
```

> Webpack has used the babel-loader to transform our code before creating our output bundle

## Configuring Babel for React

first install `npm i -S react react-dom prop-types`

create top level React component `<App/>`

the problem is that **JSX** is not valid JS

we need to configure it

1. install `npm i -D @babel/preset-react`
2. add `@babel/preset-react` to presets in `webpack.config.js`

## Injecting JS bundle into HTML

install html plugin for webpack
`npm i -D html-webpack-plugin`

we create html5 boilerpalte in `./src/index.html`

then in `webpack.config.js` we add

```JS
...
plugins: [new HtmlWebpackPlugin({
      template:'./src/index.html'
  })]
...
```

this takes our template and injects it to our dist `index.html`

### Automatically update bundle with file save

in `package.json` add to `scripts:`
`"dev" : "webpack --watch --mode development"`

when running `npm run dev` our app is watching for changes and also shows human readable code

### Modularize webpack

> As our webpack.config gets more and more complicated, we have more chances that we're going to want to override things for our development build versus a production build, and vice-versa. It would be nice to maintain two separate configs without duplicating any of the settings that are shared between the two

install `npm i -D webpack-merge`

file for settings common for development and production settings: `webpack.config.base.js`

file for specific overrides in development `webpack.config.dev.js`

```JS
module.exports = merge(baseConfig, {
    mode:'development',
})
```

file for specific overrides in production `webpack.config.prod.js`

in `package.json` update scripts to point to new files

```JSON
"scripts": {
    "build": "webpack --config webpack.config.prod.js",
    "dev": "webpack --watch --config webpack.config.dev.js",
    ...
```

## Webpack: Run your APP on localhost simulated server

`npm i -D webpack-dev-server`

in `package.json` update dev in `scripts`

```JSON
"scripts": {
    ...
    "dev": "webpack-dev-server --watch --config webpack.config.dev.js",
    ...
```

in `webpack.config.dev.js` we can specify on what port to run it

```JS
devServer:{
        port: 9000
    }
```

## Webpack: Source maps

> map our source code to the code that's actually running in the browser

in `webpack.config.dev.js` we add

```JS
...
devtool:'source-map'
```

now when we debug we can see our source code in console

### Support Proposed JavaScript Features with Babel

> transpile bleeding edge syntax

we install it via `npm i -D @babel/plugin-proposal-class-properties`

## Webpack: Supporting CSS

install loaders for styles `npm i -D css-loader style-loader`

create stylesheet

import it in `index.js`

in `webpack.config.base.js` add a new `rule`

```JS
{
        test: /\.css$/,
        use: ["style-loader", "css-loader"],
        exclude: /node_modules/
      }
```

## Webpack: Hot Reload

> if we want to avoid full-reload of our application we use so called "hot reloading"
> app refreshes with our state

install it via `npm i -S react-hot-loader`

add it as a plugin in `webpack.config.base.js`

then in `package.json` add a new `script`

`"dev:hot": "webpack-dev-server --config --hot webpack.config.dev.js",`

note: it is better to seperate `dev` and `dev:hot`

### How to avoid duplicate commands in npm

in `package.json` we can specify command just like we would run in it in a console as a value of a script `name`

eg.

```json
"dev:something":"npm run dev -- --something"
```

we pass flag by showing `--` first and then adding `--flagname`

## Webpack: Analyze bundle

> figure out the size of the bundle and visualize it

`npm i -D webpack-bundle-analyzer`

add in `webpack.config.prod.js` a new plugin

```JS
module.exports = merge(baseConfig, {
    mode:'production',
    plugins: [new BundleAnalyzerPlugin()]
})
```

to show it in a static HTML
add to Function constructor option

```JS
{
      analyzerMode: 'static'
    }
```

## Webpack: Externalize Dependencies to be loaded via CDN

> in production we would like not to include React library for user to download, but we want to use CDN to deliver React library instead

in `webpack.config.prod.js` we insert to module `externals` to say what external libraries we want to use

```JS
externals: {
        react: 'React',
        'react-dom': 'ReactDOM'
    }
```

afterwards add cdn to our html template

```js
 <% if(process.env.NODE_ENV === 'production') { %>
    <script
      crossorigin
      src="https://unpkg.com/react@16/umd/react.production.min.js"
    ></script>
    <script
      crossorigin
      src="https://unpkg.com/react-dom@16/umd/react-dom.production.min.js"
    ></script>
    <% } %>
```

## Polyfills for features backward compatibality

install `npm i -S polyfill`

in `/src/index.js` import `import '@babel/polyfill';`

> shit ton of polyfills will be available

in `webpack.config.base.js` target a specific browser

in `presets` change `"@babel/preset-env"` to an array and object as options as a second parameter, specifying e.g. we want to target chrome version 68 and above

```JS
["@babel/preset-env",{
              targets: {
                  chrome: 68
              },
              useBuiltIns: 'entry'
          }]
```

### How to analyze browsers from CLI

use tool **browserslist**

run `npx browserslist "last 2 versions` to show list of browsers and their last 2 versions

#### show last 2 versions of browsers that are not dead and have over 2 procent of market share.

`npx browserslist "last 2 versions, not dead, not < 2%"`

to translate it into config in `webpack.config.base.js`

```js
{
  targets: [
    'last 2 versions',
    'not dead',
    'not < 2%'
  ]
  ...
}
```

in `targets` developer can easilly influence which browsers to support

### Use async bundle loading

> imporve page performance and load resources only when we need them

import component like this

```JS
const Warning = React.lazy(()=> import('/.Warning'));
```

to whichever component we want to import

`React.lazy()` accepts dynamic `import` function

then in our component with dynamic import we need to add

```JS
  <React.Suspense fallback={null}>
    <Warning />
  </React.Suspense>
```

in case `import()` is not supported
install plugin with

`npm i -D @babel/plugin-syntax-dynamic-import`

## Run tests with Jest

install by:

`npm i -D jest`

if we want to write a test:

files need to be in `__tests__` or named `App.spec.js` or `App.test.js`

then write tests

run by `npm test`

**NEEDS REVISION and ADDITION**

## Formating code with Prettier

> automate formating of **all** code

install by
`npm i -D prettier pretty-quick`

add `"format": "pretty-quick"` to `package.json`

then run `npm run format` to automatically format everything since the last commit

_fun fact_ : `rc` in at the end of `.somenamerc` stands for runcom which is 'run commands' dating back to UNIX

in order to format all JS files run

`npx prettier --write "**/*.js"`

to specifiy which files should be ignored (e.g. generated files)

add `.prettierignore` and add files just like in `.gitignore`

## Avoiding errors with ESLint 

`npm i -D eslint eslint-plugin-react`

add `"lint": "eslint ./"` to `package.json` in `scripts`

to configure `.eslintrc.json` run

`npx eslint --init` and go through all questions 

to fix problems read through the log 
e.g. `it` or `describe` methods are not described add `"jest":true` into environment 

