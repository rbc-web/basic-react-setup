# DIY Setup
These are the steps I took to form the basis of this setup.

<!-- TOC START min:2 max:6 link:true asterisk:false update:true -->
- [Assumptions](#assumptions)
- [Build Pipeline Overview](#build-pipeline-overview)
	- [Package Manager](#package-manager)
	- [Bundler](#bundler)
	- [Compiler](#compiler)
- [Create Project Folder](#create-project-folder)
- [Install Packages](#install-packages)
	- [React Packages](#react-packages)
	- [webpack Packages](#webpack-packages)
		- [Why webpack-cli?](#why-webpack-cli)
	- [Babel Packages](#babel-packages)
		- [Why @babel/core?](#why-babelcore)
		- [Why babel-loader?](#why-babel-loader)
		- [Why core-js@3 and regenerator-runtime?](#why-core-js3-and-regenerator-runtime)
		- [How about using @babel/polyfill for polyfills?](#how-about-using-babelpolyfill-for-polyfills)
		- [Why @babel/preset-env?](#why-babelpreset-env)
		- [Why @babel/preset-react?](#why-babelpreset-react)
- [webpack and Babel Configuration](#webpack-and-babel-configuration)
	- [webpack Config File](#webpack-config-file)
		- [Does order matter when specifying presets?](#does-order-matter-when-specifying-presets)
		- [Where can I find a list of option for babel-loader?](#where-can-i-find-a-list-of-option-for-babel-loader)
		- [Where can I find a list of options for preset-env?](#where-can-i-find-a-list-of-options-for-preset-env)
		- [What does '> 0.2%, last 2 versions, Firefox ESR' mean?](#what-does--02-last-2-versions-firefox-esr-mean)
		- [What does 'useBuiltIns': 'usage' do?](#what-does-usebuiltins-usage-do)
		- [What does corejs option do?](#what-does-corejs-option-do)
		- [Do we have to import core-js and regenerator-runtime polyfills somewhere?](#do-we-have-to-import-core-js-and-regenerator-runtime-polyfills-somewhere)
	- [Alternative Babel Configuration](#alternative-babel-configuration)
- [Using React](#using-react)
<!-- TOC END -->



## Assumptions
* Some understanding of webpack.
* Some understanding of npm.
* Some understanding of React.

## Build Pipeline Overview
### Package Manager
npm

### Bundler
[webpack](https://webpack.js.org/)

Aside bundling, webpack provides many tools that help manage your project.

Some use cases include importing css files into your js files through a loader, or extracting and bundling them into a single file while obfuscating the file through a plugin.

### Compiler
[Babel](https://babeljs.io/)

Can compiles (JS6+) to Earlier versions.

Let us use JSX (E.g \<Tag\>\<\/Tag\>) in our JS Code.


## Create Project Folder
```Shell Session
$ mkdir nameOfYourFolder
$ cd ./nameOfYourFolder
$ npm init
```

`$` indicates terminal command.

`npm init` will ask a series of questions, most can be left with their default value. But remember the name entered for the main entry point.
## Install Packages
### React Packages
```Shell Session
$ npm install --save react
$ npm install --save react-dom
```
### webpack Packages
```Shell Session
$ npm install --save-dev webpack
$ npm install --save-dev webpack-cli
```
#### Why webpack-cli?
Think webpack-cli (webpack client) as the interface provided in terminal to interact with webpack. Basically it allows us to use webpack commands in terminal.

### Babel Packages
```Shell Session
$ npm install --save-dev @babel/core
$ npm install --save-dev babel-loader
$ npm install --save core-js@3
$ npm install --save regenerator-runtime
$ npm install --save-dev @babel/preset-env
$ npm install --save-dev @babel/preset-react
```

#### Why @babel/core?
Is the transpiler/compiler.

#### Why babel-loader?
Is a webpack loader, allows us to interact with babel in our webpack config file.

#### Why core-js@3 and regenerator-runtime?
core-js and regenerator-runtime both provides pollyfills. core-js provides polyfills for features like promises or newer ECMAScript features, while regenerator-runtime provide polyfills for generators, such as async functions and by extension allows the usage of await.

As for why core-js version 3 (@3), its the latest version but you can install version 2 if you like.

```Shell Session
$ npm install --save core-js@2
```
Currently `npm install --save core-js` defaults to version 3.

These two packages are closely related to @babel/preset-env's useBuiltIns option, [more info on option](https://babeljs.io/docs/en/babel-preset-env#usebuiltins).

**Save package as a dependency, not dev-dependency.**

[Generators info mdn](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator)

**Note: Some polyfills may not be provide, for example [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)**

#### How about using @babel/polyfill for polyfills?
>As of Babel 7.4.0, [@babel/polyfill] has been deprecated in favor of directly including core-js/stable (to polyfill ECMAScript features) and regenerator-runtime/runtime (needed to use transpiled generator functions):
>```JavaScript
>import "core-js/stable";
>import "regenerator-runtime/runtime";
>```
>[Source](https://babeljs.io/docs/en/babel-polyfill)

#### Why @babel/preset-env?
We can think of this as a predefined configuration for our compiler(babel). It defines which JavaScript features are available, like import/export. Allow us to use the latest JS features.

[More Info](https://babeljs.io/docs/en/babel-preset-env)



#### Why @babel/preset-react?
Need to install to allow syntax used in react, such as JSX.

[More Info](https://babeljs.io/docs/en/babel-preset-react)

## webpack and Babel Configuration
### webpack Config File
1. Create a webpack config file.
  * ours is called `webpack.config.js`.
  * Follow the naming convention `[name].config.js`.
1. Place the following code in your webpack config file.
  ```JavaScript
  const path = require('path');

  module.exports = {
   entry: {
       nameOfYourApp: './index.js'
   },

   output: {
     filename: '[name].bundle.js',
     path: path.resolve(__dirname, 'build')
   },

   module: {
     rules:[
       {
         test: /\.js$/,
         exclude: /node_modules/,
         use: [
           {
             loader: "babel-loader",
             options: {
               presets: [
                 ['@babel/preset-env', {
                     'targets': {
                       'browsers': '> 0.2%, last 2 versions, Firefox ESR',
                     },
                     'useBuiltIns': 'usage',
                     'corejs': 3
                   }
                 ],
                 ['@babel/preset-react'],
               ],
             },
           },
         ],
       },
     ]
   }

  };
  ```
1. Create an `index.js` file.
  * Will be left blank for now.
1. In your package.json file add `"buildFront": "webpack --config ./webpack.config.js"` under scripts. It should look like this.
```JavaScript
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "buildFront": "webpack --config ./webpack.config.js"
},
```
You can change the name of the script but be consistent afterwards.

1. Run the command
  ```Shell Session
    $ npm run buildFront
  ```
  Should produce a folder called `build`, and a file called `nameOfYourApp.bundle.js` inside of the folder. Even though `index.js` is empty, we should be able to generate the bundled file without having errors when running the command.


#### Does order matter when specifying presets?
Yes, I believe last is applied first.

#### Where can I find a list of option for babel-loader?
[Preset Option](https://babeljs.io/docs/en/options#presets)

[All Options](https://babeljs.io/docs/en/options)

#### Where can I find a list of options for preset-env?
[Preset-env Options](https://babeljs.io/docs/en/babel-preset-env#options)

#### What does '> 0.2%, last 2 versions, Firefox ESR' mean?
[You can read this for more info](https://github.com/browserslist/browserslist)


#### What does 'useBuiltIns': 'usage' do?
>Adds specific imports for polyfills when they are used in each file.
[More Info](https://babeljs.io/docs/en/babel-preset-env#usebuiltins-usage)

#### What does corejs option do?
Tells us which pollyfill package to use, core-js@2 or core-js@3.

Since we installed core-js@3, we list `3` as the option's value. You can choose version 2 but would need to install that version.

This option should only have effect when using `useBuiltIns` option.

[More info](https://babeljs.io/docs/en/babel-preset-env#corejs)

#### Do we have to import core-js and regenerator-runtime polyfills somewhere?
If using `'useBuiltIns': 'usage'`, no. The importation should automatically be done.

But if using other options, directly importing core-js and regenerator-runtime or listing them as entry points in webpack may be needed.

The below quote references @babel/polyfill as the package with polyfill, but the usage of @babel/polfill has been depreciated in favor of using core-js, and regenerator-runtime.

>When used alongside @babel/preset-env,
>
>If useBuiltIns: 'usage' is specified in .babelrc then do not include @babel/polyfill in either webpack.config.js entry array nor source. Note, @babel/polyfill still needs to be installed.
>
>If useBuiltIns: 'entry' is specified in .babelrc then include @babel/polyfill at the top of the entry point to your application via require or import as discussed above.
>
>If useBuiltIns key is not specified or it is explicitly set with useBuiltIns: false in your .babelrc, add @babel/polyfill directly to the entry array in your webpack.config.js.
>
>[More Info](https://babeljs.io/docs/en/babel-polyfill#usage-in-node-browserify-webpack)

[Docs for useBuiltIns option](https://babeljs.io/docs/en/babel-preset-env#usebuiltins)

### Alternative Babel Configuration
you can use a .babelrc file instead of placing it in babel-loader.

## Using React
1. Insert the following into index.js.
  ```JavaScript
  import React from 'react';
  import ReactDOM from 'react-dom';

  function Hello(props){
    return (
      <h1>Hello, world!</h1>
    );
  }

  ReactDOM.render(
    <Hello />,
    document.getElementById('root')
  );

  ```
1. Create a index.html file, in the same level as your app.js file, not the bundled file.
1. Insert the following into index.html.
  ```HTML
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1">
      <title>Hello World</title>
    </head>
    <body>
      <div id="root"></div>
      <script src="./build/nameOfYourApp.bundle.js"></script>
    </body>
  </html>
  ```
1. Open index.html and you should see `Hello, world!`
---
