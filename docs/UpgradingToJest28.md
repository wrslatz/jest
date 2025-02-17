---
id: upgrading-to-jest28
title: From v27 to v28
---

Upgrading Jest from v27 to v28? This guide aims to help refactoring your configuration and tests.

:::info

See [changelog](https://github.com/facebook/jest/blob/main/CHANGELOG.md) for the full list of changes.

:::

## Compatibility

The supported Node versions are 12.13, 14.15, 16.13 and above.

If you plan to use type definitions of Jest (or any of its packages), make sure to install TypeScript version 4.3 or above.

## Configuration Options

### `extraGlobals`

The `extraGlobals` option was renamed to [`sandboxInjectedGlobals`](Configuration.md#sandboxinjectedglobals-arraystring):

```diff
- extraGlobals: ['Math']
+ sandboxInjectedGlobals: ['Math']
```

### `timers`

The `timers` option was renamed to [`fakeTimers`](Configuration.md#faketimers-object). See [Fake Timers](#fake-timers) section bellow for details.

### `testURL`

The `testURL` option is removed. Now you should use [`testEnvironmentOptions`](Configuration.md#testenvironmentoptions-object) to pass `url` option to JSDOM environment:

```diff
- testURL: 'https://jestjs.io'
+ testEnvironmentOptions: {
+   url: 'https://jestjs.io'
+ }
```

## Fake Timers

Fake timers were refactored to allow passing options to the underlying [`@sinonjs/fake-timers`](https://github.com/sinonjs/fake-timers).

### `fakeTimers`

The `timers` configuration option was renamed to [`fakeTimers`](Configuration.md#faketimers-object) and now takes an object with options:

```diff
- timers: 'real'
+ fakeTimers: {
+   enableGlobally: false
+ }
```

```diff
- timers: 'fake'
+ fakeTimers: {
+   enableGlobally: true
+ }
```

```diff
- timers: 'modern'
+ fakeTimers: {
+   enableGlobally: true
+ }
```

```diff
- timers: 'legacy'
+ fakeTimers: {
+   enableGlobally: true,
+   legacyFakeTimers: true
+ }
```

### `jest.useFakeTimers()`

An object with options now should be passed to [`jest.useFakeTimers()`](JestObjectAPI.md#jestusefaketimersfaketimersconfig) as well:

```diff
- jest.useFakeTimers('modern')
+ jest.useFakeTimers()
```

```diff
- jest.useFakeTimers('legacy')
+ jest.useFakeTimers({
+   legacyFakeTimers: true
+ })
```

If legacy fake timers are enabled in Jest config file, but you would like to disable them in a particular test file:

```diff
- jest.useFakeTimers('modern')
+ jest.useFakeTimers({
+   legacyFakeTimers: false
+ })
```

## Test Environment

### Custom Environment

The constructor of [test environment](Configuration.md#testenvironment-string) class now receives an object with Jest's `globalConfig` and `projectConfig` as its first argument. The second argument is now mandatory.

```diff
  class CustomEnvironment extends NodeEnvironment {
-   constructor(config) {
-     super(config);
+   constructor({globalConfig, projectConfig}, context) {
+     super({globalConfig, projectConfig}, context);
+     const config = projectConfig;
```

### `jsdom`

If you are using JSDOM [test environment](Configuration.md#testenvironment-string), `jest-environment-jsdom` package now must be installed separately:

```bash npm2yarn
npm install --save-dev jest-environment-jsdom
```

## Test Runner

If you are using Jasmine [test runner](Configuration.md#testrunner-string), `jest-jasmine2` package now must be installed separately:

```bash npm2yarn
npm install --save-dev jest-jasmine2
```

## Transformer

`process()` and `processAsync()` methods of a custom [transformer module](CodeTransformation.md) cannot return a string anymore. They must always return an object:

```diff
  process(sourceText, sourcePath, options) {
-   return `module.exports = ${JSON.stringify(path.basename(sourcePath))};`;
+   return {
+     code: `module.exports = ${JSON.stringify(path.basename(sourcePath))};`,
+   };
  }
```

## TypeScript

:::info

The TypeScript examples from this page will only work as document if you import `jest` from `'@jest/globals'`:

```ts
import {jest} from '@jest/globals';
```

:::

### `jest.fn()`

`jest.fn()` now takes only one generic type argument. See [Mock Functions API](MockFunctionAPI.md) page for more usage examples.

```diff
  import add from './add';
- const mockAdd = jest.fn<ReturnType<typeof add>, Parameters<typeof add>>();
+ const mockAdd = jest.fn<typeof add>();
```

```diff
- const mock = jest.fn<number, []>()
+ const mock = jest.fn<() => number>()
    .mockReturnValue(42)
    .mockReturnValueOnce(12);

- const asyncMock = jest.fn<Promise<string>, []>()
+ const asyncMock = jest.fn<() => Promise<string>>()
    .mockResolvedValue('default')
    .mockResolvedValueOnce('first call');
```
