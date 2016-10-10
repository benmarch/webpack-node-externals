Webpack Bower Components externals
==============================
> Easily exclude Bower components in Webpack (Forked from https://github.com/liady/webpack-node-externals)

Webpack allows you to define [*externals*](https://webpack.github.io/docs/configuration.html#externals) - modules that should not be bundled.

When bundling with Webpack for the backend - you usually don't want to bundle its `bower_components` dependencies.
This library creates an *externals* function that ignores `bower_components` when bundling in Webpack.<br/>(Inspired by the great [Backend apps with Webpack](http://jlongster.com/Backend-Apps-with-Webpack--Part-I) series)

## Quick usage
```sh
npm install webpack-bower-externals --save-dev
```

In your `webpack.config.js`:
```js
var bowerExternals = require('webpack-bower-externals');
...
module.exports = {
    ...
    target: 'umd', // in order to ignore built-in modules like path, fs, etc.
    externals: [bowerExternals()], // in order to ignore all modules in bower_components folder
    ...
};
```
And that's it. All Bower components will no longer be bundled but will be left as `require('module')`.

## Detailed overview
### Description
This library scans the `bower_components` folder for all bower_components names, and builds an *externals* function that tells Webpack not to bundle those modules, or any sub-modules of theirs.

### Configuration
This library accepts an `options` object.

#### `options.whitelist (=[])`
An array for the `externals` to whitelist, so they **will** be included in the bundle. Can accept exact strings (`'module_name'`), regex patterns (`/^module_name/`), or a function that accepts the module name and returns whether it should be included.
<br/>**Important** - if you have set aliases in your webpack config with the exact same names as modules in *bower_components*, you need to whitelist them so Webpack will know they should be bundled.

#### `options.importType (='commonjs')`
The method in which unbundled modules will be required in the code. Best to leave as `umd` for bower components.

#### `options.modulesDir (='bower_components')`
The folder in which to search for the bower components.

#### `options.modulesFromFile (=false)`
Read the modules from the `.bower.json` file instead of the `bower_components` folder.

#### Example
```js
var bowerExternals = require('webpack-bower-externals');
...
module.exports = {
    ...
    target: 'umd', // important in order not to bundle built-in modules like path, fs, etc.
    externals: [bowerExternals({
        // this WILL include `jquery` and `webpack/hot/dev-server` in the bundle, as well as `lodash/*`
        whitelist: ['jquery', 'webpack/hot/dev-server', /^lodash/]
    })],
    ...
};
```
    
For most use cases, the defaults of `importType` and `modulesDir` should be used.

## Q&A
#### Why not just use a regex in the Webpack config?
Webpack allows inserting [regex](https://webpack.github.io/docs/configuration.html#externals) in the *externals* array, to capture non-relative modules:
```js
{
    externals: [
        // Every non-relative module is external
        // abc -> require("abc")
        /^[a-z\-0-9]+$/
    ]
}
```
However, this will leave unbundled **all non-relative requires**, so it does not account for aliases that may be defined in webpack itself.
This library scans the `bower_components` folder, so it only leaves unbundled the actual Bower components that are being used.

#### How can I bundle required assets (i.e css files) from bower_components?
Using the `whitelist` option, this is possible. We can simply tell Webpack to bundle all files with extensions that are not js/jsx/json, using this [regex](https://regexper.com/#%5C.(%3F!(%3F%3Ajs%7Cjson)%24).%7B1%2C5%7D%24):
```js
...
bowerExternals({
  // load non-javascript files with extensions, presumably via loaders
  whitelist: [/\.(?!(?:jsx?|json)$).{1,5}$/i],
}),
...
```
Thanks @wmertens for this idea.

## Contribute
Contributions and pull requests are welcome. Please run the tests to make sure nothing breaks.
### Test
```sh
npm run test
```

## License
MIT
