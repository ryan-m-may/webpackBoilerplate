# Webpack 5 Boilerplate

The intent of this project is to familiarize myself with how webpack works by diving into the documentation and setting up the best webpack boilerplate I can. This is also an exercise in autonomy and self-study.

I haven't found much *good* content online regarding webpack, it seems most tutorials and blogs revolve around `create-react-app`, so I'm using this readme to create my own lightweight documentation for future reference. That's what we do as software developers, right?

If it doesn't exist, make it! If it exists, but you don't like it, make something better!

&nbsp;
# Rules:

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

## asset/resource
Type `asset/resource` includes the file in the bundled html as a path.
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
Type `asset/inline` includes the file in the bundled html as inline html.
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

## asset/source
Type `asset/source` converts content of file into a JavaScript `string` and injects it into the bundle as is.
* This is useful for `.txt` files.
```
rules: [
  {
    tests: /\.txt$/,
    type: 'asset/source'
  }
]
```


&nbsp;
# Loaders
While Webpack includes asset loaders out of the box, any additional loaders must be installed as dependencies to the application. Multiple loaders can be included in a single rule.

## CSS
* `css-loader` only reads and returns contents of css file.
* `style-loader` injects css into page using style guides. Bundles it with JavaScript in `bundle.js`.

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
&nbsp;
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
plugins: [
  new TerserPlugin()
]
```
---

## mini-css-extract-plugin
Create a separate CSS file rather than bundling it with the JS file like `style-loader` does.
* Builds on top of a Webpack 5 feature, thus Webpack 5 is required for this plugin to work.
* Recommended to combine `mini-css-extract-plugin` with `css-loader`.
* Needs to be added to the CSS rule:
```
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
* Needs to be added to plugins.
* Output filename can be set like so:
```
plugins: [
  new MiniCssExtractPlugin({
    filename: 'styles.css',
  })
]
```
---

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
---