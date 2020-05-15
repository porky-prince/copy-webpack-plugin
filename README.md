<div align="center">
  <a href="https://github.com/webpack/webpack">
    <img width="200" height="200"
      src="https://webpack.js.org/assets/icon-square-big.svg">
  </a>
</div>

[![npm][npm]][npm-url]
[![node][node]][node-url]
[![deps][deps]][deps-url]
[![tests][tests]][tests-url]
[![cover][cover]][cover-url]
[![chat][chat]][chat-url]
[![size][size]][size-url]

# copy-webpack-plugin

Copies individual files or entire directories, which already exist, to the build directory.

## Getting Started

To begin, you'll need to install `copy-webpack-plugin`:

```console
$ npm install copy-webpack-plugin --save-dev
```

Then add the plugin to your `webpack` config. For example:

**webpack.config.js**

```js
const CopyPlugin = require('copy-webpack-plugin');

module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        { from: 'source', to: 'dest' },
        { from: 'other', to: 'public' },
      ],
    }),
  ],
};
```

> ℹ️ `webpack-copy-plugin` is not designed to copy files generated from the build process; rather, it is to copy files that already exist in the source tree, as part of the build process.

> ℹ️ If you want `webpack-dev-server` to write files to the output directory during development, you can force it with the [`writeToDisk`](https://github.com/webpack/webpack-dev-middleware#writetodisk) option or the [`write-file-webpack-plugin`](https://github.com/gajus/write-file-webpack-plugin).

## Options

The plugin's signature:

**webpack.config.js**

```js
const CopyPlugin = require('copy-webpack-plugin');

module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        { from: 'source', to: 'dest' },
        { from: 'other', to: 'public' },
      ],
      options: {
        concurrency: 100,
      },
    }),
  ],
};
```

### Patterns

|                  Name                   |        Type         |                     Default                     | Description                                                                                           |
| :-------------------------------------: | :-----------------: | :---------------------------------------------: | :---------------------------------------------------------------------------------------------------- |
|             [`from`](#from)             |     `{String}`      |                   `undefined`                   | Glob or path from where we сopy files.                                                                |
|               [`to`](#to)               |     `{String}`      |            `compiler.options.output`            | Output path.                                                                                          |
|          [`context`](#context)          |     `{String}`      | `options.context \|\| compiler.options.context` | A path that determines how to interpret the `from` path.                                              |
|      [`globOptions`](#globoptions)      |     `{Object}`      |                   `undefined`                   | [Options][glob-options] passed to the glob pattern matching library including `ignore` option.        |
|           [`toType`](#totype)           |     `{String}`      |                   `undefined`                   | Determinate what is `to` option - directory, file or template.                                        |
|            [`force`](#force)            |     `{Boolean}`     |                     `false`                     | Overwrites files already in `compilation.assets` (usually added by other plugins/loaders).            |
|          [`flatten`](#flatten)          |     `{Boolean}`     |                     `false`                     | Removes all directory references and only copies file names.                                          |
|        [`transform`](#transform)        |    `{Function}`     |                   `undefined`                   | Allows to modify the file contents.                                                                   |
|   [`cacheTransform`](#cacheTransform)   | `{Boolean\|Object}` |                     `false`                     | Enable `transform` caching. You can use `{ cache: { key: 'my-cache-key' } }` to invalidate the cache. |
|    [`transformPath`](#transformpath)    |    `{Function}`     |                   `undefined`                   | Allows to modify the writing path.                                                                    |
| [`noErrorOnMissing`](#noerroronmissing) |     `{Boolean}`     |                     `false`                     | Doesn't generate an error on missing file(s).                                                         |

#### `from`

Type: `String`
Default: `undefined`

Glob or path from where we сopy files.
Globs accept [fast-glob pattern-syntax](https://github.com/mrmlnc/fast-glob#pattern-syntax).
Glob can only be a `string`.

> ⚠️ Don't use directly `\\` in `from` option if it is a `glob` (i.e `path\to\file.ext`) option because on UNIX the backslash is a valid character inside a path component, i.e., it's not a separator.
> On Windows, the forward slash and the backward slash are both separators.
> Instead please use `/`.

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        'relative/path/to/file.ext',
        'relative/path/to/dir',
        path.resolve(__dirname, 'src', 'file.ext'),
        path.resolve(__dirname, 'src', 'dir'),
        '**/*',
        {
          from: '**/*',
        },
        // If absolute path is a `glob` we replace backslashes with forward slashes, because only forward slashes can be used in the `glob`
        path.posix.join(
          path.resolve(__dirname, 'src').replace(/\\/g, '/'),
          '*.txt'
        ),
      ],
    }),
  ],
};
```

##### `For windows`

If you define `from` as absolute file path or absolute folder path on `Windows`, you can use windows path segment (`\\`)

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: path.resolve(__dirname, 'file.txt'),
        },
      ],
    }),
  ],
};
```

But you should always use forward-slashes in `glob` expressions
See [fast-glob manual](https://github.com/mrmlnc/fast-glob#how-to-write-patterns-on-windows).

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          // If absolute path is a `glob` we replace backslashes with forward slashes, because only forward slashes can be used in the `glob`
          from: path.posix.join(
            path.resolve(__dirname, 'fixtures').replace(/\\/g, '/'),
            '*.txt'
          ),
        },
      ],
    }),
  ],
};
```

The `context` behaves differently depending on what the `from` is (`glob`, `file` or `dir`).
More [`examples`](#examples)

#### `to`

Type: `String`
Default: `compiler.options.output`

Output path.

> ⚠️ Don't use directly `\\` in `to` (i.e `path\to\dest`) option because on UNIX the backslash is a valid character inside a path component, i.e., it's not a separator.
> On Windows, the forward slash and the backward slash are both separators.
> Instead please use `/` or `path` methods.

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: '**/*',
          to: 'relative/path/to/dest/',
        },
        {
          from: '**/*',
          to: '/absolute/path/to/dest/',
        },
        {
          from: '**/*',
          to: '[path][name].[contenthash].[ext]',
        },
      ],
    }),
  ],
};
```

#### `context`

Type: `String`
Default: `options.context|compiler.options.context`

A path that determines how to interpret the `from` path.

> ⚠️ Don't use directly `\\` in `context` (i.e `path\to\context`) option because on UNIX the backslash is a valid character inside a path component, i.e., it's not a separator.
> On Windows, the forward slash and the backward slash are both separators.
> Instead please use `/` or `path` methods.

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: 'src/*.txt',
          to: 'dest/',
          context: 'app/',
        },
      ],
    }),
  ],
};
```

The `context` option can be an absolute or relative path. If `context` is a relative, then it is converted to absolute based to `compiler.options.context`

Also, `context` indicates how to interpret the search results. Further, he is considered in this role.

To determine the structure from which the found resources will be copied to the destination folder, the `context` option is used.

If `from` is a file, then `context` is equal to the directory in which this file is located. Accordingly, the result will be only the file name.

If `from` is a directory, then `context` is the same as `from` and is equal to the directory itself. In this case, the result will be a hierarchical structure of the found folders and files relative to the specified directory.

If `from` is a glob, then regardless of the `context` option, the result will be the structure specified in the `from` option

More [`examples`](#examples)

#### `globOptions`

Type: `Object`
Default: `undefined`

Allows to configute the glob pattern matching library used by the plugin. [See the list of supported options][glob-options]
To exclude files from the selection, you should use [globOptions.ignore option](https://github.com/mrmlnc/fast-glob#ignore)

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: 'public/**/*',
          globOptions: {
            dot: true,
            gitignore: true,
            ignore: ['**/file.*', '**/ignored-directory/**'],
          },
        },
      ],
    }),
  ],
};
```

#### `toType`

Type: `String`
Default: `undefined`

Determinate what is `to` option - directory, file or template.
Sometimes it is hard to say what is `to`, example `path/to/dir-with.ext`.
If you want to copy files in directory you need use `dir` option.
We try to automatically determine the `type` so you most likely do not need this option.

|       Name       |    Type    |   Default   | Description                                                                                        |
| :--------------: | :--------: | :---------: | :------------------------------------------------------------------------------------------------- |
|   **`'dir'`**    | `{String}` | `undefined` | If `to` has no extension or ends on `'/'`                                                          |
|   **`'file'`**   | `{String}` | `undefined` | If `to` is not a directory and is not a template                                                   |
| **`'template'`** | `{String}` | `undefined` | If `to` contains [a template pattern](https://github.com/webpack-contrib/file-loader#placeholders) |

##### `'dir'`

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: 'path/to/file.txt',
          to: 'directory/with/extension.ext',
          toType: 'dir',
        },
      ],
    }),
  ],
};
```

##### `'file'`

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: 'path/to/file.txt',
          to: 'file/without/extension',
          toType: 'file',
        },
      ],
    }),
  ],
};
```

##### `'template'`

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: 'src/',
          to: 'dest/[name].[hash].[ext]',
          toType: 'template',
        },
      ],
    }),
  ],
};
```

#### `force`

Type: `Boolean`
Default: `false`

Overwrites files already in `compilation.assets` (usually added by other plugins/loaders).

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: 'src/**/*',
          to: 'dest/',
          force: true,
        },
      ],
    }),
  ],
};
```

#### `flatten`

Type: `Boolean`
Default: `false`

Removes all directory references and only copies file names.

> ⚠️ If files have the same name, the result is non-deterministic.

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: 'src/**/*',
          to: 'dest/',
          flatten: true,
        },
      ],
    }),
  ],
};
```

#### `transform`

Type: `Function`
Default: `undefined`

Allows to modify the file contents.

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: 'src/*.png',
          to: 'dest/',
          // The `content` argument is a [`Buffer`](https://nodejs.org/api/buffer.html) object, it could be converted to a `String` to be processed using `content.toString()`
          // The `absoluteFrom` argument is a `String`, it is absolute path from where the file is being copied
          transform(content, absoluteFrom) {
            return optimize(content);
          },
        },
      ],
    }),
  ],
};
```

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: 'src/*.png',
          to: 'dest/',
          transform(content, path) {
            return Promise.resolve(optimize(content));
          },
        },
      ],
    }),
  ],
};
```

#### `cacheTransform`

Type: `Boolean|Object`
Default: `false`

Enable/disable `transform` caching. You can use `{ cacheTransform: { key: 'my-cache-key' } }` to invalidate the cache.
Default path to cache directory: `node_modules/.cache/copy-webpack-plugin`.

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: 'src/*.png',
          to: 'dest/',
          transform(content, path) {
            return optimize(content);
          },
          cacheTransform: true,
        },
      ],
    }),
  ],
};
```

#### `transformPath`

Type: `Function`
Default: `undefined`

Allows to modify the writing path.

> ⚠️ Don't return directly `\\` in `transformPath` (i.e `path\to\newFile`) option because on UNIX the backslash is a valid character inside a path component, i.e., it's not a separator.
> On Windows, the forward slash and the backward slash are both separators.
> Instead please use `/` or `path` methods.

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: 'src/*.png',
          to: 'dest/',
          transformPath(targetPath, absolutePath) {
            return 'newPath';
          },
        },
      ],
    }),
  ],
};
```

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: 'src/*.png',
          to: 'dest/',
          transformPath(targetPath, absolutePath) {
            return Promise.resolve('newPath');
          },
        },
      ],
    }),
  ],
};
```

### `noErrorOnMissing`

Type: `Boolean`
Default: `false`

Doesn't generate an error on missing file(s);

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: path.resolve(__dirname, 'missing-file.txt'),
          noErrorOnMissing: true,
        },
      ],
    }),
  ],
};
```

### Options

|             Name              |    Type    | Default | Description                                      |
| :---------------------------: | :--------: | :-----: | :----------------------------------------------- |
| [`concurrency`](#concurrency) | `{Number}` |  `100`  | Limits the number of simultaneous requests to fs |

#### `concurrency`

limits the number of simultaneous requests to fs

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [...patterns],
      options: { concurrency: 50 },
    }),
  ],
};
```

### Examples

#### Different variants `from` (`glob`, `file` or `dir`).

Take for example the following file structure:

```
src/directory-nested/deep-nested/deepnested-file.txt
src/directory-nested/nested-file.txt
```

##### From is a Glob

Everything that you specify in `from` will be included in the result:

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: 'src/directory-nested/**/*',
        },
      ],
    }),
  ],
};
```

Result:

```txt
src/directory-nested/deep-nested/deepnested-file.txt,
src/directory-nested/nested-file.txt
```

If you want only content `src/directory-nested/`, you should only indicate `glob` in `from`. The path to the folder in which the search should take place, should be moved to `context`.

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: '**/*',
          context: path.resolve(__dirname, 'src', 'directory-nested'),
        },
      ],
    }),
  ],
};
```

Result:

```txt
deep-nested/deepnested-file.txt,
nested-file.txt
```

##### From is a Dir

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: path.resolve(__dirname, 'src', 'directory-nested'),
        },
      ],
    }),
  ],
};
```

Result:

```txt
deep-nested/deepnested-file.txt,
nested-file.txt
```

Technically, this is `**/*` with a predefined context equal to the specified directory.

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: '**/*',
          context: path.resolve(__dirname, 'src', 'directory-nested'),
        },
      ],
    }),
  ],
};
```

Result:

```txt
deep-nested/deepnested-file.txt,
nested-file.txt
```

##### From is a File

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: path.resolve(
            __dirname,
            'src',
            'directory-nested',
            'nested-file.txt'
          ),
        },
      ],
    }),
  ],
};
```

Result:

```txt
nested-file.txt
```

Technically, this is a filename with a predefined context equal to `path.dirname(pathToFile)`.

**webpack.config.js**

```js
module.exports = {
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: 'nested-file.txt',
          context: path.resolve(__dirname, 'src', 'directory-nested'),
        },
      ],
    }),
  ],
};
```

Result:

```txt
nested-file.txt
```

## Contributing

Please take a moment to read our contributing guidelines if you haven't yet done so.

[CONTRIBUTING](./.github/CONTRIBUTING.md)

## License

[MIT](./LICENSE)

[npm]: https://img.shields.io/npm/v/copy-webpack-plugin.svg
[npm-url]: https://npmjs.com/package/copy-webpack-plugin
[node]: https://img.shields.io/node/v/copy-webpack-plugin.svg
[node-url]: https://nodejs.org
[deps]: https://david-dm.org/webpack-contrib/copy-webpack-plugin.svg
[deps-url]: https://david-dm.org/webpack-contrib/copy-webpack-plugin
[tests]: https://github.com/webpack-contrib/copy-webpack-plugin/workflows/copy-webpack-plugin/badge.svg
[tests-url]: https://github.com/webpack-contrib/copy-webpack-plugin/actions
[cover]: https://codecov.io/gh/webpack-contrib/copy-webpack-plugin/branch/master/graph/badge.svg
[cover-url]: https://codecov.io/gh/webpack-contrib/copy-webpack-plugin
[chat]: https://img.shields.io/badge/gitter-webpack%2Fwebpack-brightgreen.svg
[chat-url]: https://gitter.im/webpack/webpack
[size]: https://packagephobia.now.sh/badge?p=copy-webpack-plugin
[size-url]: https://packagephobia.now.sh/result?p=copy-webpack-plugin
[glob-options]: https://github.com/sindresorhus/globby#options
