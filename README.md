# Webpack Boilerplate

Webpack has been notoriously difficult for me to learn, so this project is a way for me to learn how to use it properly and create a little boilerplate and guide for myself.

# Rules:

Rules must be placed inside an array within `module` like so:
```
module: {
  rules: []
}
```
---

## `asset/resource`
Type `asset/resource` includes the file in the bundled html as a path.
* This is great for importing large files like a JPG because it allows the browser to download it separately. This keeps the bundle size from bloating.
```
rules: [
  {
    test: /\.(png|jpg)$/,
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
* `asset/inline` chosen if file size is less than 8kb.
* `asset/resource` chosen if file size is greater than 8kb.
* 8kb is the default setting and can be changed by using the `parser` setting as demonstrated below.
```
rules: [
  {
    test: /\.svg/,
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
    tests: /\.txt/,
    type: 'asset/source'
  }
]
```