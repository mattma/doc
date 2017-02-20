## Type Basic

- **number, string, boolean, array, any**

```ts
let decimal: number = 6;
let color: string = "blue";

let list: number[] = [1, 2, 3];

// Generic array type `Array<elemType>`
let list: Array<number> = [1, 2, 3];
```

- **void**: the opposite of `any`, the absence of having any type at all.

```ts
let unusable: void = undefined;

function warnUser(): void {
  alert("This is my warning message");
}
```

- **Null, Undefined**

By default null and undefined are subtypes of all other types. However, when using the `--strictNullChecks` flag, `null` and `undefined` are only assignable to void and their respective types. This helps avoid many common errors.

```ts
// Not much else we can assign to these variables!
let u: undefined = undefined;
let n: null = null;
```

- **never**: represents the type of values that never occur.

`never` is the return type for a function expression or an arrow function expression that always throws an exception or one that never returns; Variables also acquire the type `never` when narrowed by any type guards that can never be true.

```ts
// Function returning never must have unreachable end point
function error(message: string): never {
  throw new Error(message);
}

// Inferred return type is never
function fail() {
  return error("Something failed");
}

// Function returning never must have unreachable end point
function infiniteLoop(): never {
  while (true) { }
}
```

- **Tuple**: express an array where the type of a fixed number of elements is known, but need not be the same

```ts
let x: [string, number];  // Declare a tuple type
// Initialize it
x = ["hello", 10]; // OK
// Initialize it incorrectly
x = [10, "hello"]; // Error
```

- **Enum**: friendly names to sets of numeric values

```ts
enum Color {Red, Green, Blue};
let c: Color = Color.Green;
```

By default, enums begin numbering their members starting at 0. You can change this by manually setting the value of one of its members.

```ts
enum Color {Red = 1, Green, Blue};  // start at 1 instead of 0:

enum Color {Red = 1, Green = 2, Blue = 4};  // set all the values in the enum
let c: Color = Color.Green;
```

A handy feature of enums is to go from a numeric value to the name of that value in the enum.

```ts
enum Color {Red = 1, Green, Blue};
let colorName: string = Color[2];
alert(colorName);  // Green
```

- **readonly, ReadonlyArray<T>**

Variables use `const` whereas properties use `readonly`.

`readonly` is indicated by some properties should only be modifiable when an object is first created. TypeScript comes with a `ReadonlyArray<T>` type that is the same as `Array<T>` with all mutating methods removed, so you can make sure you don’t change your arrays after creation.

```ts
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;
ro[0] = 12; // error!
ro.push(5); // error!
ro.length = 100; // error!
a = ro; // error!

// override it with a type assertion, though, it is ok
a = ro as number[];
```


## Interfaces

A.K.A "duck typing" or "structural subtyping".

```ts
interface LabelledValue {
  label: string;
  width?: number;  // optional property

  readonly x: number;  // only be modifiable when an object is first created

  (source: string, subString: string): boolean; // function type, a call signature

  // string index signature, may have any number of other properties
  // it will make sure `label` property has a type 'string'
  [propName: string]: any;
}
```

A class has two types: the type of the static side and the type of the instance side. Ex: a construct signature and try to create a class that implements this interface you get an error:

```js
// this is an ERROR example: Because when a class implements an interface, only the instance side of the class is checked. Since the constructor sits in the static side, it is not included in this check.
interface ClockConstructor {
  new (hour: number, minute: number);
}
class Clock implements ClockConstructor {
  currentTime: Date;
  constructor(h: number, m: number) { }
}
```

To fix, you would need to work with the static side of the class directly. In this example, we define two interfaces, ClockConstructor for the constructor and ClockInterface for the instance methods. Then for convenience we define a constructor function createClock that creates instances of the type that is passed to it.

```js
interface ClockConstructor {
    new (hour: number, minute: number): ClockInterface;
}
interface ClockInterface {
    tick();
}

function createClock(ctor: ClockConstructor, hour: number, minute: number): ClockInterface {
    return new ctor(hour, minute);
}

class DigitalClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("beep beep");
    }
}
class AnalogClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("tick tock");
    }
}
// first parameter is of type ClockConstructor, it checks that AnalogClock has the correct constructor signature.
let digital = createClock(DigitalClock, 12, 17);
let analog = createClock(AnalogClock, 7, 32);
```

- **Indexable Types**: describe types “index into” like a[10], or ageMap["daniel"]. Indexable types have an index signature that describes the types we can use to index into the object, along with the corresponding return types when indexing.

```ts
interface StringArray {
  [index: number]: string;
}

// index signature states that when a StringArray is indexed with a number, it will return a string.
let myArray: StringArray= ["Bob", "Fred"];
let myStr: string = myArray[0];
```

There are two types of supported index signatures: string and number.

```ts
class Animal {
  name: string;
}
class Dog extends Animal {
  breed: string;
}
// Error: indexing with a 'string' will sometimes get you a Dog!
interface NotOkay {
  [x: number]: Animal;
  [x: string]: Dog;
}

// another example
interface NumberDictionary {
  [index: string]: number;
  length: number;    // ok, length is a number
  name: string;      // error, the type of 'name' is not a subtype of the indexer
}

// Prevent assignment to their indice, can’t set myArray[2] because the index signature is readonly
interface ReadonlyStringArray {
  readonly [index: number]: string;
}
let myArray: ReadonlyStringArray = ["Alice", "Bob"];
myArray[2] = "Mallory"; // error!
```

- **Type assertions**: like type cast

It tells the compiler "trust me, I know what I’m doing.”, and it performs no special checking or restructuring of data. It has no runtime impact, and is used purely by the compiler.

```ts
let someValue: any = "this is a string";

// "angle-braket" syntax
let strLength: number = (<string>someValue).length;

// "as" syntax
let strLength: number = (someValue as string).length;
```

A string index signature if you’re sure that the object can have some extra properties that are used in some special way. so `SquareConfig` can have any number of properties, and as long as they aren’t color property, color property is string type.

```js
interface SquareConfig {
  color?: string;
  [propName: string]: any;
}
```

- **function type** : give the interface a call signature. This is like a function declaration with only the parameter list and return type given. Each parameter in the parameter list requires both name and type.

```ts
interface SearchFunc {
  (source: string, subString: string): boolean; // function type, a call signature
}

let mySearch: SearchFunc = function(source: string, subString: string): boolean {
  let result = source.search(subString);
  return result > -1;
}
```

### Extend


## Special Cases

- Destructuring

```ts
let [first, ...rest] = [1, 2, 3, 4]; // first: 1, rest: [2, 3, 4]
let [, second, , fourth] = [1, 2, 3, 4];

function f([first, second]: [number, number]) { }

let { a: newName1, b: newName2 } = o; // Property renaming, give different name instead of "a"
let { a, b }: { a: string, b: number } = o; // Type is written separately

function keepWholeObject(wholeObject: { a: string, b?: number }) {
  let { a, b = 1001 } = wholeObject;
}
```

- Function declarations

```ts
type C = { a: string, b?: number }
function f({ a, b }: C): void { }

function f({ a, b } = { a: "", b: 0 }): void { }
f(); // ok, default to { a: "", b: 0 }
```


