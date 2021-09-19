# Webpack 5 Boilerplate

The intent of this project is to familiarize myself with how webpack works by diving into the documentation and setting up the best webpack boilerplate I can. This is also an exercise in autonomy and self-study.

I haven't found much *good* content online regarding webpack, it seems most tutorials and blogs revolve around `create-react-app`, so I'm using this readme to create my own lightweight documentation for future reference. That's what we do as software developers, right?

If it doesn't exist, make it! If it exists, but you don't like it, make something better!

&nbsp;
# Table of Contents

## [Getting Started](https://github.com/ryan-m-may/webpackBoilerplate#getting-started-1)

## [Mode](https://github.com/ryan-m-may/webpackBoilerplate#mode-1)

* None
* Development
* Production

## [Webpack Dev Server](https://github.com/ryan-m-may/webpackBoilerplate#webpack-dev-server-1)

## [Rules](https://github.com/ryan-m-may/webpackBoilerplate#rules-1)

## [Asset Modules](https://github.com/ryan-m-may/webpackBoilerplate#asset-modules-1)

* asset/resource
* asset/inline
* asset/source
* asset

## [Loaders](https://github.com/ryan-m-may/webpackBoilerplate#loaders-1)

* CSS
* Babel

## [Plugins](https://github.com/ryan-m-may/webpackBoilerplate#plugins-1)

* terser-webpack-plugin
* mini-css-extract-plugin
* clean-webpack-plugin
* html-webpack-plugin

## [Browser Caching](https://github.com/ryan-m-may/webpackBoilerplate#browser-caching-1)

&nbsp;

---

&nbsp;
# Getting Started

Follow these steps in order to use this project:

1. Fork this repository.
2. Run `npm install`
3. To use webpack:
    * For development mode: `npm run dev`
    * For production mode: `npm run build`

---
&nbsp;
# Mode

Webpack allows us to set the mode. There are several reasons to change the mode:

* Defining `mode` sets `process.env.NODE_ENV` to `production` or `development` if set to the corresponding option. If none are selected, webpack defaults to `production`.

* Errors are handled differently in `production` compared to `development` mode. It's incredibly difficult to parse the bundled, minified, `production` file, but it's easy to read errors in `development` mode.

* Several plugins are included in `production` mode by default and don't need to be included in the config file. `terser-plugin` is one of these. Production mode is designed to make the codebase streamlined, lightweight and small.

* In `development` mode, certain plugins are unnecessary. Code minification and hashing of filenames is unnecessary because users will never be accessing or caching development builds. Development mode is designed to make the build process quick and cater to developer needs rather than end user needs.

There are 3 options:
* `none`
* `development`
* `production`
```
module.exports = {
  mode: 'none',
}
```
I have built two separate webpack config files:
* `webpack.development.config.js`
* `webpack.production.config.js`

Each is designed to cater to it's corresponding environment better and will have a separate `npm` script in `package.json`.

---
&nbsp;
# Webpack Dev Server

Link to [webpack-dev-server](https://webpack.js.org/configuration/dev-server/) documentation.

Link to [webpack-dev-server](https://github.com/webpack/webpack-dev-server) github repo.

While developing new features, build time is costly. Quick build changes are valuable.

* `port`: Port where dev-server will be running.
* `static`: What will be served on that port.
* `devMiddleware`:
    * `index`: file that will be used as index file.
    * `writeToDisk`: By default, `webpack-dev-server` generates files in memory and doesn't save them to the disk. Dist will be empty, even though application is available in the browser. If set to `true`, `webpack-dev-server` writes generated files to dist directory.

```
module.exports = {
  devServer: {
    port: 3000,
    static: {
      directory: path.resolve(__dirname, './dist'),
    },
    devMiddleware: {
      index: 'index.html',
      writeToDisk: true,
    }
  }
}
```
Inside `package.json`, both `serve` and `--hot` need to be added to the `dev` script.
* `serve` is the command to start `webpack-dev-server`
* `--hot` is the command for hot module replacement. `HMR` exchanges, adds, or removes modules while an application is running, without a full reload.


```
"scripts": {
  "dev": "webpack serve --config webpack.development.config.js --hot"
}
```
>This is great for development because:
> * Webpack retains application state which is lost during a full reload.
> * Instantly update the browser when modifications are made to CSS/JS in the source code, which is almost comparable to changing styles directly in the browser's dev tools.

---
&nbsp;
# Rules

Rules must be placed inside an array within the `module` object. Individual rules will be contained inside an anonymous object.
```
module: {
  rules: [
    {
      // Rules will live here
    },
    {
      // This is how to implement a second rule
    }
  ]
}
```
---

&nbsp;
# Asset Modules

Enables asset files to be used without configuring additional loaders. Asset modules come built in with webpack 5 and things like `raw-loader`, `url-loader` and `file-loader` are unnecessary.

## asset/resource
Type `asset/resource` emits a separate file and exports the URL. Previously done with `file-loader`.

* This is great for importing large files like a JPG because it allows the browser to download it separately. This keeps the bundle size from bloating.
```
rules: [
  {
    test: /\.(png|jpg|gif)$/,
    type: 'asset/resource'
  }
]
```
---

## asset/inline
Type `asset/inline` exports a data URI of the asset. Previously done with `url-loader`.

* This is great for SVG's because since they're inline, the browser will not make separate requests for each SVG file.

* This is bad for JPG's because it will turn them to base64 in order to include them inline. This enlarges the size of the bundle dramatically.
```
rules: [
  {
    test: /\.(svg)$/,
    type: 'asset/inline'
  }
]
```
---



## asset/source
Type `asset/source` exports content of file as a JavaScript `string` and injects it into the bundle as is. Previously done with `raw-loader`.

* This is useful for `.txt` files.
```
rules: [
  {
    tests: /\.txt$/,
    type: 'asset/source'
  }
]
```
---

## asset
Type `asset` will allow webpack to make it's own decision based on file size.

* `asset/inline` is chosen if file size is less than 8kb.

* `asset/resource` is chosen if file size is greater than 8kb.

* 8kb is the default setting and can be changed by using the `parser` setting as demonstrated below.

```
rules: [
  {
    test: /\.svg$/,
    type: 'asset',
    parser: {
      dataUrlCondition: {
        maxSize: 3 * 1024 // 3kb
      }
    }
  }
]
```
---

&nbsp;
# Loaders
While Webpack includes asset loaders out of the box, any additional loaders must be installed as dependencies to the application. Multiple loaders can be included in a single rule.

## CSS
* `css-loader` only reads and returns contents of css file.
* `style-loader` injects css into page using style guides. Bundles it with JavaScript in `bundle.js`. This is recommended for `development` mode.
* `mini-css-extract-plugin` is used to create a separate file and is used in place of `style-loader` in `production` mode. Refer to the plugin section below for more details.

```
rules: [
  {
    test: /\.css$/,
    use: [
      'style-loader', 'css-loader'
    ]
  }
]
```
---

## Babel
Babel allows newer ECMAScript features to be transpiled into older versions. This allows new ECMAScript features to be used during development without compromising browser compatability during production.
* Important to exclude `node_modules`
* `@babel/env` compiles ECMAScript 6 and newer to ECMAScript 5.
* Babel plugins handle new features not covered by `@babel/env`

*Plugin/Preset order matters!*
* Plugins run before presets
* Plugin ordering is first to last
* Preset ordering is reversed (last to first)
```
rules: [
  {
    test: /\.js$/,
    exclude: /node_modules/,
    use: {
      loader: 'babel-loader',
      options: {
        presets: [ '@babel/preset-env' ],
        plugins: [ '@babel/plugin-proposal-class-properties' ]
      }
    }
  }
]
```
---
&nbsp;
# Plugins

Plugins are added inside the `plugins` array. Additional JavaScript libraries that can handle additional tasks besides loading:
* Modify how bundles are created.
* Define global constants.
* Minify bundle.
* Etc.

## terser-webpack-plugin
Webpack 5 comes with `terser-webpack-plugin` out of the box, but it must be installed in order to customize the options.
* Webpack 4 and below does not come with the plugin.
* uses `terser` to minify JavaScript.
```
const TerserPlugin = require('terser-webpack-plugin');

plugins: [
  new TerserPlugin()
]
```
---

## mini-css-extract-plugin
Create a separate CSS file rather than bundling it with the JS file like `style-loader` does.
* Recommended for `production` mode.
* Builds on top of a Webpack 5 feature, thus Webpack 5 is required for this plugin to work.
* Recommended to combine `mini-css-extract-plugin` with `css-loader`.
* Needs to be added to the CSS rule:
```
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

rules: [
  {
    test: /\.css$/,
    use: [
      MiniCssExtractPlugin.loader,
      "css-loader",
    ],
  },
],
```
* In addition to adding the rule, it also needs to be addded to plugins.
* Output filename can be set like so:
```
plugins: [
  new MiniCssExtractPlugin({
    filename: 'styles.css',
  })
]
```
---

## clean-webpack-plugin
In order to keep the `dist` directory clean, it's essential to remove files upon each build. This is especially important when using hashed names, because old files are not overwritten or saved over.

`clean-webpack-plugin` solves this problem.
```
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

plugins: [
  new CleanWebpackPlugin()
],
```
* Other directories can be cleaned with additional options.
* File paths are relative to the directory specified in the `webpack.config.js` path variable.
* In this case, that directory is `dist`
* `'**/*'` specifies that everything in dist should be cleaned.
* To clean a directory called `build` that is outside of the `dist` directory, you need the exact path like so:
```
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

plugins: [
  new CleanWebpackPlugin({
    cleanOnceBeforeBuildPatterns: [
      '**/*',
      path.join(process.cwd(), 'build/**/*')
    ]
  })
]
```
---

## html-webpack-plugin

When hashed filenames are used, filepaths can no longer be hardcoded. `html-webpack-plugin` generates a new html file that includes the correct filepaths.
```
const HtmlWebpackPlugin = require('html-webpack-plugin');

plugins: [
  new HtmlWebpackPlugin()
]
```

Additional options can be specified for this plugin to enable more control of the generated file.

[Here's a list of options included in the plugin.](https://github.com/jantimon/html-webpack-plugin#options)

And a link to the documentation for [creating and using an HTML template](https://github.com/jantimon/html-webpack-plugin/blob/main/docs/template-option.md) for even more control over the generated file

```
const HtmlWebpackPlugin = require('html-webpack-plugin');

plugins: [
  new HtmlWebpackPlugin({
    title: 'Hello World',
    filename: html/generatedHtml.html,
    meta: {
      description: 'Description'
    }
  })
]
```

---
&nbsp;
# Browser Caching

Browsers download assets before loading websites. Each time a user reloads a page, the browser downloads all of the files again. This is inefficient.

Browsers can cache files to save time. This introduces a problem: If you change a file, the user's browser won't download the new one if it has the same name, because browsers remember files by name.

To solve this, use content hash, which creates a hash based on the contents of the file.

*HOW COOL IS THAT!?*

* This is a feature included in webpack and nothing needs to be installed.
* add `[contenthash]` wherever a filename is specified, such as the `output` object:

```
output: {
  filename: 'bundle.[contenthash].js',
}
```
* Or the minified CSS file:
```
plugins: [
  new MiniCssExtractPlugin({
    filename: 'styles.[contenthash].css',
  })
],
```
There's one problem with this: now index.html doesn't know what the file names are. This is why `html-webpack-plugin` is used.

---
&nbsp;