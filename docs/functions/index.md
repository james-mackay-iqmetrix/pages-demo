# Functions

## What is a function?

We've extracted common iqmetrix functionality into a set of functions that can be imported and reused wherever they're needed. We've based the design of these functions around the principles of [SOA in the browser](https://medium.com/canopy-tax/a-case-for-soa-in-the-browser-f777a9f139b2). We have a few simple rules that define what a function in this repository looks like:

### Target an ES5 environment

This means functions will include their own polyfills and transpile down to ES5. We don't want teams needing to manage a graph of polyfill dependencies in order to use our functions.

### Export as UMD modules

This means functions will include a [UMD](https://www.davidbcalhoun.com/2014/what-is-amd-commonjs-and-umd/) module wrapper. We don't want to lock teams into a particular module-loader.

### Communicate via promises

This means functions will always return a promise, even when they're synchronous. We want teams to be able to count on a consistent return format.

```code
lang: javascript
---
export default async () => {
  return await somethingAsync();
};
```

```code
lang: javascript
---
export default () =>
  new Promise((resolve, reject) => {
    somethingAsync()
      .then(resolve)
      .catch(reject);
  });
```

### Have unbreakable interfaces

This means that once a function has been merged into master, its inputs, expected behavior, and outputs are "locked-in". We want teams to be able to lazy-load these functions at run time without inheriting breaking changes. Functions can be extended in backwards-compatible manner, but if it's a breaking change, it's a new function.

### Consume parameters via an options object

This means functions will accept a single javascript object of key:value pairs instead of an ordered list of parameters. We don't want our backwards-compatible changes to accidentally become breaking as a result of changed parameter order.

```code
lang: javascript
---
export default ({ param1, param2 }) => {
  console.log(param1);
  console.log(param2);
};
```
