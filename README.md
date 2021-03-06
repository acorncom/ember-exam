# Ember Exam

[![Build Status](https://travis-ci.org/trentmwillis/ember-exam.svg)](https://travis-ci.org/trentmwillis/ember-exam)

Ember Exam is an addon to allow you more control over how you run your tests. It provides the ability to randomize, split, distill, and parallelize* your test suite by adding a more robust CLI command.

It started as a way to help reduce flaky tests and encourage healthy test driven development. It's like [Head & Shoulders](http://www.headandshoulders.com/) for your tests!

**Note: this addon is only compatible with Ember-CLI v1.13.10 and up.**

_*As of v1.0.0 parallelization isn't ready, but it is being worked on!_

## How To Use

Using Ember Exam is fairly straightforward as it extends directly from the default Ember-CLI `test` command. So, by default, it will work exactly the same as `ember test`.

```bash
$ ember exam
$ ember exam --filter='acceptance'
$ ember exam --server
```

 The idea is that you should be able to replace `ember test` with `ember exam` and never look back.

### Filtering

**FEATURE IN PROGRESS**

```bash
$ ember exam --distill=<string|regex>
```

The default `filter` option only applies a filter in browser that is used by QUnit. This means all your tests are still loaded and parsed, but only a subset are run. Additionally, you can only use a simple string to filter on.

The `distill` option filters the tests after build so that you only load and parse those tests that are being run. It also supports taking a regular expression string to match tests against for added flexibility.

### Randomization

```bash
$ ember exam --random=<''|'tests'|'modules'>
```

By default, the `random` option allows you to run your test modules in a random order. This is essentially the same as if you were you to randomize the structure of your test directory. You can also use `--random=modules` to achieve the same effect more explicitly.

If you specify `--random=tests`, it will randomizes test modules and the tests within those modules. This means that the test modules will be randomly ordered and then the tests within them will be randomly ordered as well.

```bash
$ ember exam --random --seed=<num>
```

The `seed` option allows you to specify a starting value from which to randomize your test/module order. This allows reproduction of issues that occurred when running randomly. There are no bounds on what the seed value can be, though generated seeds are within the range of [0, 10000).

### Splitting

```bash
$ ember exam --split=<num>
```

The `split` option allows you to specify a number of files greater than one to spread your tests across. Ember will then proceed to run only the first bacth of tests.

The split will attempt to weight your test modules (by type of test module and number of tests) so that each group runs in roughly the same amount of time.

```bash
$ ember exam --split=<num> --split-file=<num>
```

The `split-file` options allows you to specify which test file to run after using the `split` option. It is one-indexed, so if you specifiy a split of 3, the highest file you could run is 3 as well.

```bash
$ ember exam --split=<num> --weighted
```

The `weighted` option splits tests by weighting them according to type; `acceptance` tests weigh more than `unit` tests weigh more than `jshint` tests. This helps make sure the various test groupings run in similar amounts of time.

**FEATURE IN PROGRESS**

```bash
$ ember exam --split=<num> --parallel
```

The `parallel` option allows you to run your split tests across multiple child processes. It can only be used when you also split your tests.

## Usage Constraints

Since Ember Exam performs many of its functions by inspecting the Abstract Syntax Tree of your tests, it is bound by some constraints. Specifically, any task that involves identifying individual tests (e.g., distilling and randomizing tests) are currently limited to tests that follow a structure similar to:

```javascript
import { module, test } from 'qunit';
module('Some Module');
test('Some Test');
```

This is because identifying a call to `test` that is actually for creating a test, does not have a 100% foolproof heuristic. So Ember Exam looks for calls that get compiled to either: `QUnit.test`, `_qunit.test`, or `_emberQunit.test`, since this covers the large majority of Ember tests and lines up with the examples given in the [Ember testing guides](http://guides.emberjs.com/v2.2.0/testing/).
