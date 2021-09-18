# Webpack 5 Boilerplate

Webpack has been notoriously difficult for me to learn, so I decided to create my own boilerplate.
I'm using this readme as a place to write some notes that will serve as lightweight documentation for future reference.

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

## `asset/resource`
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

## `asset/inline`
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

## `asset`
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

## `asset/source`
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

## `CSS`
* `css-loader` only reads and returns contents of css file.
* `style-loader` injects css into page using style guides. Bundles it with JavaScript in `bundle.js`.

```
rules: [
  {
    tests: /\.css$/,
    use: [
      'style-loader', 'css-loader'
    ]
  }
]
```
---
&nbsp;
## `Babel`
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
    tests: /\.js$/,
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