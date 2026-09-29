# @stackline/trough

> `trough` is middleware.

[![npm version](https://img.shields.io/npm/v/@stackline/trough.svg?style=flat-square)](https://www.npmjs.com/package/@stackline/trough)
[![license](https://img.shields.io/npm/l/@stackline/trough.svg?style=flat-square)](https://github.com/alexandroit/stackline-trough)
[![GitHub repository](https://img.shields.io/badge/GitHub-repository-181717?style=flat-square&logo=github)](https://github.com/alexandroit/stackline-trough)
[![Docs](https://img.shields.io/badge/docs-alexandro.net-0f766e?style=flat-square)](https://alexandro.net/docs/vanilla/trough/)
[![Reddit community](https://img.shields.io/badge/community-r%2FStackline-ff4500?style=flat-square&logo=reddit&logoColor=white)](https://www.reddit.com/r/Stackline/)

**[Documentation](https://alexandro.net/docs/vanilla/trough/)** | **[npm](https://www.npmjs.com/package/@stackline/trough)** | **[Issues](https://github.com/alexandroit/stackline-trough/issues)** | **[Repository](https://github.com/alexandroit/stackline-trough)**

**Current package version:** `1.0.2`

---

## Why this package?

`@stackline/trough` is the Stackline-maintained distribution of `trough@2.2.0`. It is an independent continuation of [trough](https://github.com/wooorm/trough); original authors and licenses remain credited below.

## Compatibility

| Item | Value |
| :--- | :--- |
| Package | `@stackline/trough@1.0.2` |
| API target | `trough@2.2.0` |
| Supported Node.js | `See supported framework requirements` |
| License | `MIT` |
| Module type | `module` |
| Types | `./index.d.ts` |
| Runtime dependencies | `none` |

## Installation

```bash
npm install @stackline/trough
```

Preserve existing imports and plugin resolution with an npm alias:

```bash
npm install trough@npm:@stackline/trough
```

## Usage and API reference

### trough


`trough` is middleware.

## Contents

*   [What is this?](#what-is-this)
*   [When should I use this?](#when-should-i-use-this)
*   [Install](#install)
*   [Use](#use)
*   [API](#api)
    *   [`trough()`](#trough-1)
    *   [`wrap(middleware, callback)`](#wrapmiddleware-callback)
    *   [`Callback`](#callback)
    *   [`Middleware`](#middleware)
    *   [`Pipeline`](#pipeline)
    *   [`Run`](#run)
    *   [`Use`](#use-1)
*   [Compatibility](#compatibility)
*   [Security](#security)
*   [Contribute](#contribute)
*   [License](#license)

## What is this?

`trough` is like [`ware`][github-segmentio-ware] with less sugar.
Middleware functions can also change the input of the next.

The word **trough** (`/trôf/`) means a channel used to convey a liquid.

## When should I use this?

You can use this package when you’re building something that accepts “plugins”,
which are functions, that can be sync or async, promises or callbacks.

## Install

This package is [ESM only][github-gist-esm].
In Node.js (version 16+),
install with [npm][npm-install]:

```sh
npm install @stackline/trough
```

In Deno with [`esm.sh`][esm-sh]:

```js
import {trough, wrap} from 'https://esm.sh/trough@2'
```

In browsers with [`esm.sh`][esm-sh]:

```html
<script type="module">
  import {trough, wrap} from 'https://esm.sh/trough@2?bundle'
</script>
```

## Use

```js
import fs from 'node:fs'
import path from 'node:path'
import process from 'node:process'
import {trough} from '@stackline/trough'

const pipeline = trough()
  .use(function (fileName) {
    console.log('Checking… ' + fileName)
  })
  .use(function (fileName) {
    return path.join(process.cwd(), fileName)
  })
  .use(function (filePath, next) {
    fs.stat(filePath, function (error, stats) {
      next(error, {filePath, stats})
    })
  })
  .use(function (ctx, next) {
    if (ctx.stats.isFile()) {
      fs.readFile(ctx.filePath, next)
    } else {
      next(new Error('Expected file'))
    }
  })

pipeline.run('readme.md', console.log)
pipeline.run('node_modules', console.log)
```

Yields:

```txt
Checking… readme.md
Checking… node_modules
Error: Expected file
    at ~/example.js:22:12
    at wrapped (~/node_modules/trough/index.js:111:16)
    at next (~/node_modules/trough/index.js:62:23)
    at done (~/node_modules/trough/index.js:145:7)
    at ~/example.js:15:7
    at FSReqCallback.oncomplete (node:fs:199:5)
null <Buffer 23 20 74 72 6f 75 67 68 0a 0a 5b 21 5b 42 75 69 6c 64 5d 5b 62 75 69 6c 64 2d 62 61 64 67 65 5d 5d 5b 62 75 69 6c 64 5d 0a 5b 21 5b 43 6f 76 65 72 61 ... 7994 more bytes>
```

## API

This package exports the identifiers
[`trough`][api-trough] and
[`wrap`][api-wrap].
There is no default export.

It exports the [TypeScript][] types
[`Callback`][api-callback],
[`Middleware`][api-middleware],
[`Pipeline`][api-pipeline],
[`Run`][api-run],
and [`Use`][api-use].

### `trough()`

Create new middleware.

###### Parameters

There are no parameters.

###### Returns

[`Pipeline`][api-pipeline].

### `wrap(middleware, callback)`

Wrap `middleware` into a uniform interface.

You can pass all input to the resulting function.
`callback` is then called with the output of `middleware`.

If `middleware` accepts more arguments than the later given in input,
an extra `done` function is passed to it after that input,
which must be called by `middleware`.

The first value in `input` is the main input value.
All other input values are the rest input values.
The values given to `callback` are the input values,
merged with every non-nullish output value.

*   if `middleware` throws an error,
    returns a promise that is rejected,
    or calls the given `done` function with an error,
    `callback` is called with that error
*   if `middleware` returns a value or returns a promise that is resolved,
    that value is the main output value
*   if `middleware` calls `done`,
    all non-nullish values except for the first one (the error) overwrite the
    output values

###### Parameters

*   `middleware` ([`Middleware`][api-middleware])
    — function to wrap
*   `callback` ([`Callback`][api-callback])
    — callback called with the output of `middleware`

###### Returns

Wrapped middleware ([`Run`][api-run]).

### `Callback`

Callback function (TypeScript type).

###### Parameters

*   `error` (`Error`, optional)
    — error, if any
*   `...output` (`Array<unknown>`, optional)
    — output values

###### Returns

Nothing (`undefined`).

### `Middleware`

A middleware function called with the output of its predecessor (TypeScript
type).

###### Synchronous

If `fn` returns or throws an error,
the pipeline fails and `done` is called with that error.

If `fn` returns a value (neither `null` nor `undefined`),
the first `input` of the next function is set to that value
(all other `input` is passed through).

The following example shows how returning an error stops the pipeline:

```js
import {trough} from '@stackline/trough'

trough()
  .use(function (thing) {
    return new Error('Got: ' + thing)
  })
  .run('some value', console.log)
```

Yields:

```txt
Error: Got: some value
    at ~/example.js:5:12
    …
```

The following example shows how throwing an error stops the pipeline:

```js
import {trough} from '@stackline/trough'

trough()
  .use(function (thing) {
    throw new Error('Got: ' + thing)
  })
  .run('more value', console.log)
```

Yields:

```txt
Error: Got: more value
    at ~/example.js:5:11
    …
```

The following example shows how the first output can be modified:

```js
import {trough} from '@stackline/trough'

trough()
  .use(function (thing) {
    return 'even ' + thing
  })
  .run('more value', 'untouched', console.log)
```

Yields:

```txt
null 'even more value' 'untouched'
```

###### Promise

If `fn` returns a promise,
and that promise rejects,
the pipeline fails and `done` is called with the rejected value.

If `fn` returns a promise,
and that promise resolves with a value (neither `null` nor `undefined`),
the first `input` of the next function is set to that value (all other `input`
is passed through).

The following example shows how rejecting a promise stops the pipeline:

```js
import {trough} from '@stackline/trough'

trough()
  .use(function (thing) {
    return new Promise(function (resolve, reject) {
      reject('Got: ' + thing)
    })
  })
  .run('thing', console.log)
```

Yields:

```txt
Got: thing
```

The following example shows how the input isn’t touched by resolving to `null`.

```js
import {trough} from '@stackline/trough'

trough()
  .use(function () {
    return new Promise(function (resolve) {
      setTimeout(function () {
        resolve(null)
      }, 100)
    })
  })
  .run('Input', console.log)
```

Yields:

```txt
null 'Input'
```

###### Asynchronous

If `fn` accepts one more argument than the given `input`,
a `next` function is given (after the input).
`next` must be called, but doesn’t have to be called async.

If `next` is given a value (neither `null` nor `undefined`) as its first
argument,
the pipeline fails and `done` is called with that value.

If `next` is given no value (either `null` or `undefined`) as the first
argument,
all following non-nullish values change the input of the following
function,
and all nullish values default to the `input`.

The following example shows how passing a first argument stops the pipeline:

```js
import {trough} from '@stackline/trough'

trough()
  .use(function (thing, next) {
    next(new Error('Got: ' + thing))
  })
  .run('thing', console.log)
```

Yields:

```txt
Error: Got: thing
    at ~/example.js:5:10
```

The following example shows how more values than the input are passed.

```js
import {trough} from '@stackline/trough'

trough()
  .use(function (thing, next) {
    setTimeout(function () {
      next(null, null, 'values')
    }, 100)
  })
  .run('some', console.log)
```

Yields:

```txt
null 'some' 'values'
```

###### Parameters

*   `...input` (`Array<any>`, optional)
    — input values

###### Returns

Output, promise, etc (`any`).

### `Pipeline`

Pipeline (TypeScript type).

###### Properties

*   `run` ([`Run`][api-run])
    — run the pipeline
*   `use` ([`Use`][api-use])
    — add middleware

### `Run`

Call all middleware (TypeScript type).

Calls `done` on completion with either an error or the output of the
last middleware.

> 👉 **Note**: as the length of input defines whether async functions get a
> `next` function,
> it’s recommended to keep `input` at one value normally.

###### Parameters

*   `...input` (`Array<any>`, optional)
    — input values
*   `done` ([`Callback`][api-callback])
    — callback called when done

###### Returns

Nothing (`undefined`).

### `Use`

Add middleware (TypeScript type).

###### Parameters

*   `middleware` ([`Middleware`][api-middleware])
    — middleware function

###### Returns

Current pipeline ([`Pipeline`][api-pipeline]).

## Compatibility

This projects is compatible with maintained versions of Node.js.

When we cut a new major release,
we drop support for unmaintained versions of Node.
This means we try to keep the current release line,
`trough@2`,
compatible with Node.js 12.

## Security

This package is safe.

## Contribute

Yes please!
See [How to Contribute to Open Source][open-source-guide-contribute].

## License

[MIT][file-license] © [Titus Wormer][wooorm]



[api-callback]: #callback

[api-middleware]: #middleware

[api-pipeline]: #pipeline

[api-run]: #run

[api-trough]: #trough

[api-use]: #use

[api-wrap]: #wrapmiddleware-callback

[badge-build-image]: https://github.com/wooorm/trough/workflows/main/badge.svg

[badge-build-url]: https://github.com/wooorm/trough/actions

[badge-coverage-image]: https://img.shields.io/codecov/c/github/wooorm/trough.svg

[badge-coverage-url]: https://codecov.io/github/wooorm/trough

[badge-downloads-image]: https://img.shields.io/npm/dm/trough.svg

[badge-downloads-url]: https://www.npmjs.com/package/trough

[badge-size-image]: https://img.shields.io/bundlejs/size/trough

[badge-size-url]: https://bundlejs.com/?q=trough

[npm-install]: https://docs.npmjs.com/cli/install

[esm-sh]: https://esm.sh

[file-license]: license

[github-gist-esm]: https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c

[github-segmentio-ware]: https://github.com/segmentio/ware

[open-source-guide-contribute]: https://opensource.guide/how-to-contribute/

[typescript]: https://www.typescriptlang.org

[wooorm]: https://wooorm.com

## Credits and original authors

- Original project: [trough](https://github.com/wooorm/trough).
- Titus Wormer.
- Copyright (c) 2016 Titus Wormer <tituswormer@gmail.com>.
- Stackline maintenance: [Alexandro Paixao Marques](https://www.linkedin.com/in/aleinfo/) and [Stackline contributors](https://github.com/alexandroit).

Original copyright, license notices and contributor acknowledgements remain part of this distribution. Stackline maintenance does not replace authorship of the original work.

## Community and Links

- [Stackline website](https://alexandro.net/)
- [GitHub projects](https://github.com/alexandroit)
- [npm packages](https://www.npmjs.com/~alex360qc)
- [Reddit community — r/Stackline](https://www.reddit.com/r/Stackline/)
- [Maintainer LinkedIn](https://www.linkedin.com/in/aleinfo/)

Use this repository's issue tracker for reproducible bugs and feature requests. Join r/Stackline for examples, usage questions and release discussions.
