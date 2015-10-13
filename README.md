# quibble

Quibble is sorta like [proxyquire](https://github.com/thlorenz/proxyquire),
[sandboxed-module](https://github.com/felixge/node-sandboxed-module) and
[mockery](https://github.com/mfncooper/mockery). Using `quibble` you can replace
how `require` will behave for a given path, with its intended use being almost
solely unit testing.

## Usage

Say we're testing pants:

```
quibble = require('quibble')

describe('pants', function(){
  var subject, legs;
  beforeEach(function(){
    legs = quibble('./../lib/legs', function(){ return 'ooh, legs';});

    subject = require('./../lib/pants');
  });
  // ... more test stuff
});
```

That way, when the `subject` loaded from `lib/pants` runs `require('./legs')`,
it will get back the function that returns 'ooh, legs'.

## Configuration

There's only one option: what you want to do with quibbled modules by default.

Say you're pulling in [testdouble.js](https://github.com/testdouble/testdouble.js)
and you want every quibbled module to default to a single test double function with
a name that matches its absolute path. You could do this:

```
quibble = require('quibble')
beforeEach(function(){
  quibble.config({
    defaultFakeCreator: function(path) {
      return require('testdouble').create(path);
    }
  });
});
```

With this set up, running `quibble('./some/path')` will default to replacing all
`require('../anything/that/matches/some/path')` invocations with a test double named
after the absolute path resolved to by `'./some/path'`.

Spiffy!

## How's it different?

A few things that stand out about quibble:

1. No partial mocking, as proxyquire does. [Partial Mocks](https://github.com/testdouble/contributing-tests/wiki/Partial-Mock)
are often seen problematic and not helpful for unit testing designed to create clear boundaries
between the SUT and its dependencies
2. Global replacements, so it's easy to set up a few arrange steps in advance of
instantiating your subject (using `require` just as you normally would). The instantiation
style of other libs is a little different (e.g. `require('./my/subject', {'/this/thing': stub})`
3. Require strings are resolved to absolute paths. It can be a bit confusing using other tools because from the perspective of the test particular paths are knocked out _from the perspective of the subject_ and not from the test listing, which runs counter to how every other Node.js API works. Instead, here, the path of the file being knocked out is relative to whoever is knocking it out.
4. A configurable default faker function. This lib was written with [testdouble.js](https://github.com/testdouble/testdouble.js) in mind, thinking that you'd want to default to just create a new testdouble and return it in each case by default, without having to manually create it and pass it in (for somewhat more minimal test setup)
5. A `reset()` method that undoes everything, intended to be run `afterEach` test runs

