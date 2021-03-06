# NodeJS interop library

[![Build Status](https://travis-ci.org/pulyaevskiy/node-interop.svg?branch=master)](https://travis-ci.org/pulyaevskiy/node-interop)

Provides interface bindings for NodeJS APIs and allows writing Dart
applications and libraries which can be compiled and used in NodeJS.

Unlock the full power of Dart static analysis, type system and built-in features
like Futures, Streams and async/await for building NodeJS apps.

## Philosophy

Provided API interface is Dart-centric which means Node functionality is 
exposed in idiomatic Dart as much as possible. For instance:

- File system API follows `dart:io` interface (thanks to the awesome `file` package)
- Platform API also exposed via interface provided by `platform` package.
- Dart's Futures and Streams are used whenever possible.

The intention is to provide familiar and productive environment for Dart
developers when working with NodeJS.

## Usage

How to create a simple Node app written in Dart:

1. Create a new Dart project. Stagehand is a great way to scaffold all the
  boilerplate.
  ```bash
  $ pub global activate stagehand
  $ mkdir my_node_app
  $ cd my_node_app
  $ stagehand package-simple # if you have .pub-cache in your PATH, or
  $ pub global run stagehand package-simple # if you don't have it in your PATH
  ```
2. Add dependency and transformers to the generated `pubspec.yaml`
  ```yaml
  dependencies:
    node_interop: ^0.0.1

  transformers:
    - $dart2js
    - node_interop
  ```
3. Create `bin/main.dart` and write some code:
  ```dart
  import 'package:node_interop/node_interop.dart';
  import 'package:node_interop/fs.dart';

  void main() async {
    var fs = new NodeFileSystem(); // access Node `fs` module
    print(fs.currentDirectory); // prints the path from `process.cwd()`
    var files = await fs.currentDirectory.list().toList();
    print(files); // lists current directory contents
  }
  ```
4. **Compile.**
  Pub by default assumes you are building for web so it looks for `web` folder. We need to explicitely tell it to look in `bin/` where our `main.dart` is:
  ```bash
  $ pub build bin
  ```
5. **Run the app.** Compiled app is located in `build/bin/main.dart.js`:
  ```bash
  $ # assuming you have NodeJS installed:
  $ node build/bin/main.dart.js
  $ # If everything worked well you should see your current 
  $ # directory contents printed out.
  ```

## Pub transformer

This library provides Pub transformer which by default processes all `.dart`
files in the `bin/` folder. The only thing it does is it prepends Node preamble
to JS generated by dart2js transformer.

## NodeJS modules and `exports`

It is possible to create a module which exposes some functionality via NodeJS
`exports`.

Here is simple module example exporting a `bang` function:

```dart
// file:bin/bang.dart
import 'package:node_interop/node_interop.dart';

/// A module which exports a bang function.
void main() {
  exports.setProperty('bang', bang);
}

String bang(String value) {
  return value.toUpperCase() + '!';
}
```

After compiled with `pub build` it can be `require`d from any JS file:

```js
const bang = require('path/to/build/bin/bang.dart.js');
console.log(bang.bang('Hi')); 
// prints out "HI!"
```

> For such a small function the resulting `bang.dart.js` would be quite
> large (~1k lines of code). However creating Node modules like this can
> be considered an edge use case when you would want to mix in some 
> functionality from Dart into an existing Node app.

## Features and bugs

Please file feature requests and bugs at the [issue tracker](http://github.com/pulyaevskiy/node-interop/issues/new)
