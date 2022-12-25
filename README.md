# @indutny/sneequals

[![npm](https://img.shields.io/npm/v/@indutny/sneequals)](https://www.npmjs.com/package/@indutny/sneequals)
[![size](https://img.shields.io/bundlephobia/minzip/@indutny/sneequals)](https://bundlephobia.com/result?p=@indutny/sneequals)
![CI Status](https://github.com/indutny/sneequals/actions/workflows/test.yml/badge.svg)

Sneaky equals comparison between objects that checks only the properties that
were touched.

Inspired by [proxy-compare](https://github.com/dai-shi/proxy-compare).

## Installation

```sh
npm install @indutny/sneequals
```

## Usage

### One object comparison

```js
import { watch } from '@indutny/sneequals';

const originalData = {
  nested: {
    prop: 1,
  },
  avatar: {
    src: 'image.png',
  },
};

const { proxy, watcher } = watch(originalData);

function doSomethingWithData(data) {
  return {
    prop: data.nested.prop,
    x: data.avatar,
  };
}

const result = watcher.unwrap(doSomethingWithData(proxy));

// Prevent further access to proxy
watcher.stop();

const sneakyEqualData = {
  nested: {
    prop: 1,
    other: 'ignored',
  },
  avatar: original.avatar,
};

console.log(watcher.isChanged(originalData, sneakyEqualData)); // false

const sneakyDifferentData = {
  nested: {
    prop: 2,
  },
  avatar: {
    ...original.avatar,
  },
};

console.log(watcher.isChanged(originalData, sneakyDifferentData)); // true
```

### Multi object comparison

```js
import { watchAll } from '@indutny/sneequals';

const inputA = { a: 1 };
const inputB = { b: 2 };

const { proxies, watcher } = watchAll([inputA, inputB]);

function fn(a, b) {
  return a.a + a.b;
}

const result = watcher.unwrap(fn(...proxies));

// Prevent further access to proxies
watcher.stop();

console.log(watcher.isChanged(inputA, { a: 1 })); // false
console.log(watcher.isChanged(inputB, { b: 3 })); // true
```

### Memoization

`memoize()` is provided as a convenience API method. It has a simple
WeakMap-based cache that keys by first object argument of the function and/or
global internal key.

```js
import { memoize } from '@indutny/sneequals';

const fn = memoize((a, b) => {
  return a.a + a.b;
});
```

## Benchmarks

On M1 Macbook Pro 13:

```sh
% npm run bench

> @indutny/sneequals@1.3.5 bench
> bencher dist/benchmarks/*.js

isChanged: 12494398.2 ops/s (±18229.0, p=0.05, n=95)
isNotChanged: 11131421.5 ops/s (±50286.6, p=0.05, n=99)
memoize: 4595006.7 ops/s (±4717.3, p=0.05, n=97)
watch+unwrap: 865008.6 ops/s (±6086.9, p=0.05, n=98)
```

## Credits

Name coined by [Scott Nonnenberg](https://github.com/scottnonnenberg/).

## LICENSE

This software is licensed under the MIT License.
