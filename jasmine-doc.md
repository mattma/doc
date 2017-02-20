# [Jasmine](https://jasmine.github.io)

## [Introduction](https://jasmine.github.io/2.5/introduction)

`describe` and `it` blocks are just functions, they can contain any executable code necessary to implement the test.

### Suites: `describe`, `xdescribe`

A test suite (your tests for grouping related specs) begins with a call to the global Jasmine function describe with two parameters: a string ( A spec suite name/title) and a function (code implements the suite).

`describe` can be nested, with specs defined at any level. This allows a suite to be composed as a tree of functions. Before a spec is executed, Jasmine walks down the tree executing each `beforeEach` function in order. After the spec is executed, Jasmine walks through the `afterEach` functions similarly.

 `xdescribe`: disabling suites. These suites and any specs inside them are skipped when run and thus their results will show as pending.

### Specs: `it`, `xit`

Specs are defined by calling the global Jasmine function `it`. The string is the title of the spec and the function is the spec, or test. A spec contains one or more expectations that test the state of the code.

An expectation in Jasmine is an assertion that is either true or false. A spec with all true expectations is a **passing spec**. A spec with one or more false expectations is a **failing spec**.

- Pending specs

`xit`: Pending specs do not run, but their names will show up in the results as pending.

Any spec declared without a function body will also be marked pending in results. ex: `it("can be declared with 'it' but without a function");`

`pending` function, if defined anywhere in the spec body, no matter the expectations, the spec will be marked pending. A string passed to pending will be treated as a reason and displayed when the suite finishes.

### Expectations: `expect`, `fail`

Expectations are built with the function `expect` which takes a value, called the actual. It is chained with a Matcher function, which takes the expected value.

`fail`: manually failing a spec. The fail function causes a spec to fail. It can take a failure message or an Error object as a parameter.

### Setup and Teardown

Jasmine provides the global `beforeEach`, `afterEach`, `beforeAll`, and `afterAll` functions.

`beforeEach` function is called once before each spec in the describe in which it is called.
`afterEach` function is called once after each spec.
`beforeAll` function is called only once before all the specs in describe are run.
`afterAll` function is called after all specs finish.

`beforeAll` & `afterAll` can be used to speed up test suites with expensive setup and teardown. However, Since they are not reset between specs, it is easy to accidentally leak state between your specs so that they erroneously pass or fail.

### `this` keyword

Share variables between a `beforeEach`, `it`, etc is through the `this` keyword. Each spec's `beforeEach/it/afterEach` has the `this` as the same empty object that is set back to empty for the next spec's `beforeEach/it/afterEach`.

- Example

```js
describe("A suite", function() {
  var foo = 0;

  beforeEach(() => { foo += 1;  this.bar = 0; });
  afterEach(() => foo = 0); // `this.baz` has been reset to empty

  it("is just a function, so it can contain any code, even multiple expectations", function() {
    expect(foo).toEqual(1);
    expect(this.bar).toEqual(0);
    this.baz = "test pollution?";
  });

  it("should not call the callBack", function() {
    expect(this.baz).toBe(undefined);
    var foo = (x, callBack) => if (x) callBack();
    foo(false, () => fail("Callback has been called"));
  });

  // without a function body will also be marked pending in results
  it("can be declared with 'it' but without a function");

  it("can be declared by calling 'pending' in the spec body", function() {
    expect(true).toBe(false);
    pending('this is why it is pending');
  });
});
```

### Matchers

Each matcher implements a boolean comparison (ex: `.toBe`) between the actual value and the expected value. It is responsible for reporting to Jasmine if the expectation is true or false. Jasmine will then pass or fail the spec. ex: `expect(true).toBe(true);`

Any matcher can evaluate to a negative assertion by chaining the call to `expect` with a `not` before calling the matcher. ex: `expect(false).not.toBe(true);`

- Included Matchers

**toBe**, **toEqual**, **toMatch**, **toBeDefined**, **toBeUndefined**, **toBeNull**, **toBeTruthy**, **toBeFalsy**, **toContain**, **toBeLessThan**, **toBeGreaterThan**, **toBeCloseTo**, **toThrow**, **toThrowError**

```js
// `toBe` matcher compares with ===
expect(a).toBe(b);
expect(a).not.toBe(null);

// `toEqual` match simple literals, variables, objects
expect({a: 12}).toEqual({a: 12});

// `toMatch` matcher is for regular expressions
expect(message).toMatch(/bar/);
expect(message).toMatch("bar");
expect(message).not.toMatch(/quux/);

// `toBeDefined` matcher compares against `undefined`
expect(a.foo).toBeDefined();
expect(a.bar).not.toBeDefined();

// `toBeUndefined` matcher compares against `undefined`
expect(a.foo).not.toBeUndefined();
expect(a.bar).toBeUndefined();

// `toBeNull` matcher compares against null
expect(null).toBeNull();
expect(foo).not.toBeNull();

// `toBeTruthy` matcher is for boolean casting testing
expect(foo).toBeTruthy();
expect(a).not.toBeTruthy();

// `toBeFalsy` matcher is for boolean casting testing
expect(a).toBeFalsy();
expect(foo).not.toBeFalsy();

// `toContain` matcher: in array, or finding a substring
var a = ["foo", "bar", "baz"];  // var a = "foo bar baz";
expect(a).toContain("bar");
expect(a).not.toContain("quux");

// `toBeLessThan` matcher is for mathematical comparisons
var pi = 3.1415926, e = 2.78;
expect(e).toBeLessThan(pi);
expect(pi).not.toBeLessThan(e);

// `toBeGreaterThan` matcher is for mathematical comparisons
expect(pi).toBeGreaterThan(e);
expect(e).not.toBeGreaterThan(pi);

// `toBeCloseTo` matcher is for precision math comparison
expect(pi).not.toBeCloseTo(e, 2);
expect(pi).toBeCloseTo(e, 0);

// `toThrow` matcher is for testing if a function throws an exception
var foo = () => 1 + 2;
var bar = () => a + 1;
var baz = () => throw 'what';
expect(foo).not.toThrow();
expect(bar).toThrow();
expect(baz).toThrow('what');

// `toThrowError` matcher is for testing a specific thrown exception
var foo = () => throw new TypeError("foo bar baz");
expect(foo).toThrowError("foo bar baz");
expect(foo).toThrowError(/bar/);
expect(foo).toThrowError(TypeError);
expect(foo).toThrowError(TypeError, "foo bar baz");
```

- Custom Matchers

## Spies

They are test double functions. A spy can stub any function and tracks calls to it and all arguments. A spy only exists in the `describe` or `it` block in which it is defined, and will be removed after each spec. There are special matchers for interacting with spies.

### Matchers

```js
describe("A spy", function() {
  var foo, bar = null;

  beforeEach(function() {
    foo = {
      setBar: function(value) {
        bar = value;
      }
    };

    spyOn(foo, 'setBar');

    foo.setBar(123);
    foo.setBar(456, 'another param');
  });

  // The toHaveBeenCalled matcher will return true if the spy was called.
  it("tracks that the spy was called", function() {
    expect(foo.setBar).toHaveBeenCalled();
  });

  // The toHaveBeenCalledTimes matcher will pass if the spy was called the specified number of times.
  it("tracks that the spy was called x times", function() {
    expect(foo.setBar).toHaveBeenCalledTimes(2);
  });
¶
  //  toHaveBeenCalledWith matcher will return true if the argument list matches any of the recorded calls to the spy.
  it("tracks all the arguments of its calls", function() {
    expect(foo.setBar).toHaveBeenCalledWith(123);
    expect(foo.setBar).toHaveBeenCalledWith(456, 'another param');
  });

  it("stops all execution on a function", function() {
    expect(bar).toBeNull();
  });
});
```





