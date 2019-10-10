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

* we can run webpack from node_modules `node_modules/.bin/webpack`
* better way is to configure script in `"script"` within `package.json` 

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
const path = require("path");

module.exports = {
  entry: "./src/index.js",
  output: {
    //   __dirname is global variable in node that gives us our current directory 
    path: path.join(__dirname, "dist"),
    filename: "app.bundle.js"
  }
};
```

**important note:** webpack uses node process and therefore uses `require()` to import modules 

>The code intended to run in a browser uses ES module syntax and is transformed with Babel into code that will work in a browser environment
