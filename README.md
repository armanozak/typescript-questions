# TypeScript Questions

Back in April and May 2023, I asked daily TypeScript questions on [my Twitter account](https://twitter.com/armanozak) for 30 working days. This repository is originally a compilation of these questions, but if you want to contribute, please feel free to create a PR.

## Q1

```typescript
const foo = "1234";
```

What is the type of `foo`?

<details>
  <summary>Answer</summary>

`"1234"`

TypeScript infers this type as a string literal and not as string because it is a constant. If we had used let, then the inferred type would have been string.
  
Can we force TypeScript to use `string` here? Yes.
  
```typescript
const foo: string = "1234"
```
  
Info: https://typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types

</details>

## Q2

```typescript
enum Flag {
  Foo,
  Bar = -2,
  Baz
}
```

What are the values of `Foo` and `Baz` flags?

> Note: This is just an example to test your knowledge. I recommend assigning all enum values explicitly, even when they are ordered.

<details>
  <summary>Answer</summary>

**Foo = 0, Baz = -1**

When an enum member doesn't have an initializer, it is assigned as previous member value +1 or 0 if it is the first member.

Info: https://typescriptlang.org/docs/handbook/enums.html#computed-and-constant-members

</details>

## Q3

```typescript
const myObject = {
  id: crypto.randomUUID(),
};

type Id =                        ;
//                  ^
// What should I write here to create a type alias
// for the id property of myObject?
```

I want to create a type alias for `myObject`'s `id` property. How should I fill in the blank to do this?

<details>
  <summary>Answer</summary>

`typeof myObject.i​d`

We want to keep the `Id` type alias compatible with the actual type of that property, so we can't;

- use hard-coded types.
- leverage any technique that gets the return type of randomUUID (because it can change too).

Info: https://typescriptlang.org/docs/handbook/2/typeof-types.html

</details>

## Q4

```typescript
function add(x: number, y: number) {
  return x + y;
}

const params = [1, 2];

add(...params);
//      ^
// I'm getting an error here. How can I make it go away?
```

I want to spread those parameters, but I'm getting an error. What should I do?

<details>
  <summary>Answer</summary>

```typescript
const params: Parameters<typeof add> = [1, 2];
```

Can't we use a tuple instead? Yes, assigning `[number, number]` to or using `as const` on params will work too. However, deriving the type from the parameters of the `add` function will always keep them in sync.

Info: https://typescriptlang.org/docs/handbook/utility-types.html#parameterstype

</details>

## Q5

```typescript
interface Foo {
  x: number;
  y: number;
}

interface Bar {
  q: string;
}

let fooOrBar: Foo | Bar;

fooOrBar = { q: "" };
type T1 = typeof fooOrBar;

fooOrBar = { x: 0, y: 1 };
type T2 = typeof fooOrBar;
```

What are `T1` and `T2`?

<details>
  <summary>Answer</summary>

**T1 = Bar, T2 = Foo**

The type of `fooOrBar` is a union type (`Foo | Bar`). However, TypeScript is able to narrow types based on assignments. So, `T1` becomes Bar. Then we assign another value and TypeScript narrows the type again, this time as `Foo`.

Info: https://typescriptlang.org/docs/handbook/2/narrowing.html#assignments

</details>

## Q6

```typescript
class SplitAfterEach {
  constructor(public readonly search: string) {}

  [Symbol.split](str: string) {
    const result: string[] = [];

    do {
      let index = str.indexOf(this.search);
      if (index < 0) {
        result.push(str);
        break;
      }

      index += this.search.length;
      result.push(str.substring(0, index));
      str = str.substring(index);
    } while (str.length);

    return result;
  }
}

console.log("foobarfoobar".split(new SplitAfterEach("foo")));

type StringKeysOfSplitAfterEach =                              ;
//                                             ^
// How can I get only the string keys of SplitAfterEach?
```

How should I fill in the blank to get only the string keys of `SplitAfterEach`?

<details>
  <summary>Answer</summary>

**Option 1:** `keyof SplitAfterEach & string`

We can get keys with `keyof`, but keys can be of `number` or `symbol` type too. An intersection type narrows it for us.

Info: https://typescriptlang.org/docs/handbook/2/objects.html#intersection-types

**Option 2:** Use a utility type

`Extract<keyof SplitAfterEach, string>`

Info: https://typescriptlang.org/docs/handbook/utility-types.html#extracttype-union

</details>

## Q7

```typescript
type X = Awaited<Promise<Promise<string>>>;
```

What is `X`?

<details>
  <summary>Answer</summary>

`string`

`Awaited` is a utility type that unwraps promises recursively.

Info: https://typescriptlang.org/docs/handbook/utility-types.html#awaitedtype

</details>

## Q8

```typescript
type BuildEvent = "BuildError" | "BuildSuccess";
type HttpEvent = "HttpError" | "HttpSuccess";
type InitEvent = "InitError" | "InitSuccess";
type NavEvent = "NavError" | "NavSuccess";

type AppEvent = BuildEvent | HttpEvent | InitEvent | NavEvent;

type AppError = Extract<AppEvent,                   >;
//                                        ^
// What should I write here to get all error types?
// i.e. "BuildError" | "HttpError" | "InitError" | "NavError"
```

How should I fill in the blank to get all error types and none of the success types?

<details>
  <summary>Answer</summary>

`${string}Error`

Template literals allow us to compose string literal types, and string acts like a wildcard. So, when `Extract` asserts the assignability of the `AppEvent` union against `${string}Error`, all members ending with "Error" pass.

Info: https://typescriptlang.org/docs/handbook/2/template-literal-types.html

</details>

## Q9

```typescript
type BuildEvent = "BuildError" | "BuildSuccess";
type HttpEvent = "HttpError" | "HttpSuccess";
type InitEvent = "InitError" | "InitSuccess";
type NavEvent = "NavError" | "NavSuccess";

type AppEvent = BuildEvent | HttpEvent | InitEvent | NavEvent;

// This works. The type is a union of all errors.
type AppError = Extract<AppEvent, `${string}Error`>;

// This doesn't! The type is never.
type AppNever = AppEvent extends `${string}Error` ? AppEvent : never;

// Why?
```

In the last question, we used `Extract` to get all error types. [`Extract` is a very simple generic type](https://github.com/microsoft/TypeScript/blob/main/src/lib/es5.d.ts):

`type Extract<T, U> = T extends U ? T : never;`

However, when I implement the same conditional type directly, it doesn't work. Why?

<details>
  <summary>Answer</summary>

Generic types distribute unions.

Every member of a union is evaluated separately when passed to any generic type. So this works too:

```typescript
type ExtractError<T> = T extends `${string}Error` ? T : never;
type AppError = ExtractError<AppEvent>;
```

Info: https://typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types

</details>

## Q10

```typescript
// human.ts
export class Human {
  constructor(public readonly name: string) {}

  breath() {
    console.log(`${this.name} breaths`);
  }

  eat(food: string) {
    console.log(`${this.name} eats ${food}`);
  }

  sleep() {
    console.log(`${this.name} sleeps`);
  }

  walk() {
    console.log(`${this.name} walks`);
  }
}
```

```typescript
// speech.ts
import { Human } from "./human";

Human.prototype.speak = function (this: Human, sth: string) {
  //              ^
  // I'm getting an error: Property 'speak' does not exist on type 'Human'.

  console.log(`${this.name} says "${sth}"`);
};


// How can I add the "speak" method the "Human" interface?
```

```typescript
// index.ts
import { Human } from "./human";
import "./speech";

const human = new Human("Guru");

human.speak("Hello world!");
```

How can I add a `speak` method to the `Human` interface using the _speech_ module?

<details>
  <summary>Answer</summary>

With module augmentation.

Placing the code below in _speech.ts_ will add the speak method to the `Human` interface:

```typescript
declare module './human' {
  interface Human {
    speak(str: string): void;
  }
}
```

Info: https://typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation

</details>

## Q11

```typescript
/* Note: This file is imported by the entry file. */

import * as MSW from "msw";

(global as any).msw = MSW;

// How can I add "msw" to the global scope in TypeScript?
```

I want to use the MSW library in my test files without importing it in each file, so I mutated the global object. How can I add `msw` to the global scope in TypeScript too?

<details>
  <summary>Answer</summary>

With global augmentation.

Adding this to the _test-globals.ts_ file will declare `msw` as a global variable:

```typescript
declare global {
  const msw: typeof MSW;
}
```

Info: https://typescriptlang.org/docs/handbook/declaration-merging.html#global-augmentation

</details>

## Q12

```typescript
type MapLink = string;

interface Address {
  street: string;
  no: string;
  postalCode: string;
  city: string;
  country: string;
}

interface Coordinates {
  x: number;
  y: number;
}

declare function getLinkForAddress(address: Address): MapLink;
declare function getLinkForCoordinates(coordinates: Coordinates): MapLink;

function getLink(addressOrCoordinates: Address | Coordinates) {
  if (isCoordinates(addressOrCoordinates)) {
    //      ^
    // This doesn't narrow the type to Coordinates and I get errors.
    return getLinkForCoordinates(addressOrCoordinates);
  }

  return getLinkForAddress(addressOrCoordinates);
}

// What should I add to isCoordinates function to narrow the type?
function isCoordinates(input: object) {
  return "x" in input && "y" in input;
}
```

I am getting errors when I pass `addressOrCoordinates` as an argument to `getLinkForCoordinates` and `getLinkForAddress` functions. What should I add to `isCoordinates` function to narrow the type?

<details>
  <summary>Answer</summary>

`input is Coordinates` as return type

```typescript
function isCoordinates(
  input: object
): input is Coordinates {
  return "x" in input && "y" in input;
}
```

This kind of return type is called a "type predicate".

Info: https://typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates

</details>

## Q13

```typescript
interface Petrol {
  engineType: "combustion";
  fillTank(): void;
}

interface Diesel {
  engineType: "combustion";
  fillTank(): void;
}

interface Electric {
  engineType: "electricity";
  charge(): void;
}

interface Hybrid {
  engineType: "combustion & electricity";
  fillTank(): void;
  charge(): void;
}

type CarType = Petrol | Diesel | Electric | Hybrid;

function getArea(car: CarType) {
  switch (car.engineType) {
    case "combustion":
      return car.fillTank();
    case "electricity":
      return car.charge();
    default:
      return exhaustCases(car);
      //                   ^
      // How can I declare this function so that I get a type error here?
  }
}
```

I want to get a type error when not all cases are exhausted. How should I declare the `exhaustCases` function for that? Please use the `declare function ...` syntax.

<details>
  <summary>Answer</summary>

```typescript
declare function exhaustCases(
  input: never
): never;
```

`never` is assignable to all types, but no other type is assignable to `never`. If the case clauses in your switch statement exhaust all members of the union type, TypeScript will narrow the type in the default clause down to `never`, so there will be no errors. In this example, the `Hybrid` interface is not exhausted, so we would get an error if we try to assign it to a parameter with `never` type.

Info: https://typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking

> Why is the return type `never` and not `void`? 🤔
> 
> Well, in this example, it is possible to return void. However, we would have to update it if we ever update the return types of `fillTank` and `charge` methods, otherwise we would end up with a union type for no reason. Take a look at this playground: https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgAoTFA9gG2QbwChlkIQBzUCAFQE8AHCALmQCIEsBbAIwFcBnMMCwhWAbmLIYwHDmpwQAawAUAShbcsuCAokBfQoVCRYiFABFgEfhDxESZSiBoNmbDjwFCR4ydNnySmoaWjg6IPqGxtDwSMgAomEImMAIBJKOVHSMLKy2EMlQqcBgtL4kCAAWcFDkEMHImtq6hAZG4DFmyAAStNxFACbpDhRZrrkefILCIMgAZKRJKQglZRIk-nIKKuqNoeHryFU1dQ1NYS1tpYzIAMI12SgAvGgY2HgAPsiW1rbIX4kCst-j0+oMJIQYLwQMkZsgoBAYAj+JV4s5arQAMpYXhQJDKBA1Fj3KCPVTIAD0ACo9s1Zl8AG5YYBDKkU4bIfgAdxKVWQBJqADpMs4yRyKnAbO4uFNvKImJISCQEWBcbNCVBBZtAjtDhKpXklkUVqVWAqlUqVWqjkLjrV6qo9cgBoi4LwcGBzRb4RhrRAAB7VLz3Gz8AVQR2SAxtF0IHA1FBQmFy0iBt2CEPWZSgei8T3IZwM6C7JkssRAA
> 
> `never` as return type is the best option because it won't change the return type of the function and will always work. So, we can even have a common `exhaustCases` function that works for all switch statements.

</details>

## Q14

```typescript
type Operation = "upsert" | "delete";

interface Options {
  skipErrors?: boolean;
}

// None of these should lead to type errors:
execute("upsert", "foo");
execute("delete", "foo", { skipErrors: false });
execute("upsert", "foo", "bar");
execute("delete", "foo", "bar", { skipErrors: true });
execute("upsert", "foo", "bar", "baz");
execute("delete", "foo", "bar", "baz", {});
execute("upsert", ...(["foo", "bar"] as const));
execute("delete", ...(["foo", "bar"] as const), {});
execute("upsert", "x", ...["foo", "bar", "baz"]);
execute("delete", "x", ...["foo", "bar", "baz"], {});

// All of these should lead to type errors:
execute("upsert");
execute("delete", {});
execute("upsert", {}, "foo");
execute("delete", 1, "foo");
execute("upsert", "foo", 1, "bar");
execute("delete", "foo", { skipErrors: "true" });
execute("foo", "bar", "baz");

// How should I declare the execute function?
// Note: Use void as return type.
```

How should I declare the `execute` function? You can use `void` as its return type. Please use the `declare function ...` syntax.

<details>
  <summary>Answer</summary>

```typescript
declare function execute(
  operation: Operation,
  requiredParam: string,
  ...optionalParams: string[] | [...string[], Options]
): void;
```

Using an overload here is worse than a union type because of two reasons:

- Instead of just "skipErrors", TypeScript auto-complete displays all string methods with an "s".
- `execute("upsert", "foo", 1, "bar")` is completely highlighted as an error because it doesn't match any overloads, whereas with union types only the part that doesn't match the signature is highlighted.

> Please try not to declare a function like that in your work. 😅

</details>

## Q15

```typescript
type A = void extends true ? 1 : 0;
type B = true extends void ? 1 : 0;
type X = (() => void) extends (() => true) ? 1 : 0;
type Y = (() => true) extends (() => void) ? 1 : 0;
```

What are `A`, `B`, `X`, and `Y`?

<details>
  <summary>Answer</summary>

**A = 0, B = 0, X = 0, Y = 1**

`void` and `true` don't extend each other, but `void` as a return type has a special status to allow this:

```typescript
declare const records: string[];
declare function register(rec: string): boolean;
records.forEach(register);
```

Info: https://typescriptlang.org/docs/handbook/2/functions.html#return-type-void

</details>

## Q16

```typescript
class Point {
  get #self() {
    return this as Mutable<Point>;
  }

  constructor(
    public readonly x: number,
    public readonly y: number
  ) {}

  move(x: number, y: number) {
    this.#self.x += x;
    this.#self.y += y;
  }

  [Symbol.iterator]() {
    return [this.x, this.y][Symbol.iterator]();
  }
}

const point = new Point(0, 1);
console.log(...point); // 0 1

point.move(2, 3);
console.log(...point); // 2 4
```

Please declare the `Mutable` type and make sure it is reusable.

<details>
  <summary>Answer</summary>

```typescript
type Mutable<T> = {
  -readonly [K in keyof T]: T[K];
};
```

Removing modifiers like `readonly` and optional (`?`) is possible while mapping types.

Info: https://typescriptlang.org/docs/handbook/2/mapped-types.html#mapping-modifiers

</details>

## Q17

```typescript
interface Point {
  readonly x: number;
  readonly y: number;
}

type Mutable<T, Keys extends keyof T = keyof T> = Reveal<
  {
    -readonly [K in keyof Pick<T, Keys>]: T[K];
  } & Omit<T, Keys>
>;

type HorizontallyMovablePoint = Mutable<Point, "x">;
// {
//   x: number;
//   readonly y: number;
// }
```

Let's add the option to pick keys to the `Mutable` type in the previous question. What should the `Reveal` type be, if we want to see the whole interface and not some obscure intersection type on hover?

<details>
  <summary>Answer</summary>

```typescript
type Reveal<T> = {
  [K in keyof T]: T[K];
} & {};
```

</details>

## Q18

```typescript
interface Point {
  [stringIndex: string]: any;
  [symbolIndex: symbol]: any;
  [templateLiteralIndex: `distanceTo${string}`]: number;
  name: string;
  readonly x: number;
  readonly y: number;
  move(x: number, y: number): void;
}

type BasePoint = OmitIndexSignatures<Point>;
// type BasePoint = {
//   name: string;
//   readonly x: number;
//   readonly y: number;
//   move: (x: number, y: number) => void;
// }
```

How should I declare the `OmitIndexSignatures` type so that `BasePoint` won't have any index signatures?

Ref: https://typescriptlang.org/docs/handbook/2/objects.html#index-signatures

<details>
  <summary>Answer</summary>

```typescript
type OmitIndexSignatures<T> = {
  [
    K in keyof T
      as {} extends Record<K, 1> ? never : K
  ]: T[K];
};
```

Credit: https://stackoverflow.com/questions/51465182/how-to-remove-index-signature-using-mapped-types/68261113#68261113

</details>

## Q19

```typescript
type T0 = Await<Promise<boolean>>; // boolean
type T1 = Await<Promise<Promise<number>>>; // number
type T2 = Await<Promise<Promise<Promise<symbol>>>>; // symbol
type T3 = Await<Pick<Promise<string>, 'then'>>; // string
type T4 = Await<{then<T>(mapFn: (url: URL) => T, flag: boolean): T}>; // URL
type T5 = Await<{then(): Promise<URL>}>; // never
type T6 = Await<{then: Event}>; // { then: Event }
type T7 = Await<Date>; // Date
type T8 = Await<null>; // null
type T9 = Await<undefined>; // undefined
```

Please declare the `Await` type without using (or peeking at 🙂) the built-in `Awaited` type. You may assume strict mode.

<details>
  <summary>Answer</summary>

```typescript
type Await<T> =
  T extends {
    then: (fn: infer Fn, ...args: any[]) => any
  }
    ? Fn extends (value: infer V, ...args: any[]) => any
        ? Await<V>
        : never
    : T;
```

Conditional types allow us to infer types with the `infer` keyword.

Info: https://typescriptlang.org/docs/handbook/2/conditional-types.html#inferring-within-conditional-types

</details>

## Q20

```typescript
interface Component {
  on_mount(): void;
  on_update(): void;
  on_destroy(): void;
  parent_ref?: Component;
}

type HookName = ParseHookNames<Component>;
//      ^
// "Mount" | "Update" | "Destroy"
```

Please declare the `ParseHookNames` type.

<details>
  <summary>Answer</summary>

```typescript
type ParseHookName<T> =
  T extends `on_${infer K}`
    ? Capitalize<K>
    : never;

type ParseHookNames<T> = ParseHookName<keyof T>;
```

Info: https://typescriptlang.org/docs/handbook/2/template-literal-types.html#inference-with-template-literals

</details>

## Q21

```typescript
abstract class Child {
  play() {};
}

abstract class Adult {
  doWhatYouWant() {};
  haveFun() {};
  spendTimeWithLovedOnes() {};
  rest() {};
}

class Parent extends Adult {
  constructor(public children: [Child, ...Child[]]) {
    super();
  }
}

class WorkingAdult extends Working(Adult) {}

class WorkingParent extends Working(Parent) {
  haveSomeTimeOff() {
    this.children.forEach(child => child.play());
    return super.haveSomeTimeOff();
  }
}

function spendToday(workers: WorkingAdult[]): boolean {
  return workers.every(person => person.haveSomeTimeOff());
}
```

Please declare the `Working` function.

<details>
  <summary>Answer</summary>

```typescript
type AbstractConstructor<T> = abstract new (...args: any[]) => T;
type AbstractClass<T> = AbstractConstructor<T> & { prototype: T };

interface WorkingImpl {
  work(): void; // just for presentational purposes
  haveSomeTimeOff(): boolean;
}

declare function Working<T extends AbstractConstructor<Adult>>(
  Base: T
): AbstractClass<WorkingImpl> & T;
```

Info: https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-2.html#abstract-construct-signatures

</details>

## Q22

```typescript
declare const uniqueSymbol: unique symbol;
interface Foo { n: number; }
interface Bar { n: number; }
interface Baz { n: number; a: boolean; }

type T01 = IsSame<null, null>; // true
type T02 = IsSame<undefined, undefined>; // true
type T03 = IsSame<void, void>; // true
type T04 = IsSame<null, undefined>; // false
type T05 = IsSame<null, void>; // false
type T06 = IsSame<void, null>; // false
type T07 = IsSame<undefined, void>; // false
type T08 = IsSame<void, undefined>; // false
type T09 = IsSame<true, boolean>; // false
type T10 = IsSame<number, number>; // true
type T11 = IsSame<number, 0>; // false
type T12 = IsSame<1, number>; // false
type T13 = IsSame<symbol, symbol>; // true
type T14 = IsSame<symbol, typeof uniqueSymbol>; // false
type T15 = IsSame<string, string>; // true
type T17 = IsSame<string, ''>; // false
type T18 = IsSame<'X', string>; // false
type T19 = IsSame<string, Promise<string>>; // false
type T20 = IsSame<Promise<string>, Promise<string>>; // true
type T21 = IsSame<Date, Date>; // true
type T22 = IsSame<object, Date>; // false
type T23 = IsSame<object, object>; // true
type T24 = IsSame<{}, object>; // false
type T25 = IsSame<{}, {}>; // true
type T26 = IsSame<Foo, {}>; // false
type T27 = IsSame<Foo, Bar>; // true
type T28 = IsSame<Foo, {n: number}>; // true
type T29 = IsSame<Foo, Baz>; // false
type T30 = IsSame<Baz, Foo>; // false
```

Please declare the `IsSame` type. We should be able to switch the two generic variable types and get the same result.

<details>
  <summary>Answer</summary>

```typescript
type IsSame<T1, T2> =
  (<T>() => (T extends T1 ? 1 : 0)) extends
  (<T>() => (T extends T2 ? 1 : 0))
    ? true
    : false;
```

Credit: https://github.com/microsoft/TypeScript/issues/27024#issuecomment-421529650

How does it work? Here's a comment from the TypeScript codebase:

> Two conditional types 'T1 extends U1 ? X1 : Y1' and 'T2 extends U2 ? X2 : Y2' are related if one of T1 and T2 is related to the other, U1 and U2 are identical types, X1 is related to X2, and Y1 is related to Y2.

Let's rewrite our type to observe the resemblance:

```typescript
type IsSame<U1, U2> =
  (<T1>() => (T1 extends U1 ? 'X' : 'Y')) extends
  (<T2>() => (T2 extends U2 ? 'X' : 'Y'))
    ? true
    : false;
```

To resolve the outer conditional type, TypeScript checks if `U1` and `U2` are identical, and that's exactly what we need. 🤷‍♂️

</details>

## Q23

```typescript
interface Point {
  x: number;
  y: number;
  name?: string;
}

interface Circle {
  center: Point;
  radius: number;
  name?: string;
}

type RequiredKeysOfPoint = RequiredKeysOf<Point>;
// "x" | "y"

type RequiredKeysOfCircle = RequiredKeysOf<Circle>;
// "center" | "radius"
```

Please declare the "RequiredKeysOf" type and make sure it is reusable.

<details>
  <summary>Answer</summary>

```typescript
type RequiredKeysOf<T> = RequiredKeysOf1<OmitIndexSignatures<T>>;

type RequiredKeysOf1<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? never : K;
} [keyof T];

type RequiredKeysOf2<T> = {
  [K in keyof T]-?: T extends Record<K, T[K]> ? K : never;
}[keyof T];

type RequiredKeysOf3<T> = keyof {
  [K in keyof T as {} extends Pick<T, K> ? never : K]: unknown;
};

type RequiredKeysOf4<T> = keyof {
[K in keyof T as T extends Record<K, T[K]> ? K : never]: unknown;
};

type OmitIndexSignatures<T> = {
  [K in keyof T as {} extends Record<K, 1> ? never : K]: T[K];
}
```

All 4 options work as long as index signatures are removed. Options 1 & 2 reveal the keys even when all keys are required, whereas options 3 & 4 return `keyof T` (e.g. `keyof Point`).

</details>

## Q24

```typescript
interface Bank { code: string; }
interface Currency { code: string; sign: string; }

type OpaqueBank = Opaque<Bank, "BANK">;
type OpaqueCurrency = Opaque<Currency, "CURRENCY">;
type OpaqueIBAN = Opaque<string, "IBAN">;

interface BankAccount {
  bank: OpaqueBank;
  currency: OpaqueCurrency;
  holder: string;
  iban: OpaqueIBAN;
}

declare const account: BankAccount;

// Assigning a value directly to "bank", "currency" or "iban" should be an error.
account.bank = { code: "TCZBTR2AXXX" }; // Error
account.bank = account.currency; // Error
account.currency = { code: "USD", sign: "$" }; // Error
account.iban = "TR320010009999901234567890"; // Error

// But, it should be possible to use them as their base type.
declare function formatCurrency(amount: number, currency: Currency): string;
const formattedAmount = formatCurrency(128_000_000_000, account.currency);

declare function formatIBAN(iban: string): string;
const formattedIBAN = formatIBAN(account.iban);

// This should be "bank" | "currency" | "iban".
type OpaqueKeysOfBankAccount = OpaqueKeysOf<BankAccount>;

// This should be "BANK" | "CURRENCY" | "IBAN".
type OpaqueTokensOfBankAccount = OpaqueTokensOf<BankAccount>;

// And, these should be allowed.
(account.bank as Transparent<OpaqueBank>) = { code: "TCZBTR2AXXX" };
(account.currency as Transparent<OpaqueCurrency>) = { code: "USD", sign: "$" };
(account.iban as Transparent<OpaqueIBAN>) = "TR320010009999901234567890";
(account as ClearOpaqueKeysOf<BankAccount, "bank">).bank = { code: "TCZBTR2AXXX" };
(account as ClearOpaqueKeysOf<BankAccount, "bank" | "iban">).bank = { code: "TCZBTR2AXXX" };
(account as ClearOpaqueKeysOf<BankAccount, string>).bank = { code: "TCZBTR2AXXX" };
(account as ClearOpaqueKeysOf<BankAccount, unknown>).bank = { code: "TCZBTR2AXXX" };
(account as ClearOpaqueKeysOf<BankAccount, any>).bank = { code: "TCZBTR2AXXX" };
(account as ClearOpaqueKeysOf<BankAccount>).bank = { code: "TCZBTR2AXXX" };
(account as ClearOpaqueKeysOf<BankAccount, "currency">).currency = { code: "USD", sign: "$" };
(account as ClearOpaqueKeysOf<BankAccount, "currency" | "iban">).currency = { code: "USD", sign: "$" };
(account as ClearOpaqueKeysOf<BankAccount>).currency = { code: "USD", sign: "$" };
(account as ClearOpaqueKeysOf<BankAccount, "iban">).iban = "TR320010009999901234567890";
(account as ClearOpaqueKeysOf<BankAccount, "bank" | "iban">).iban = "TR320010009999901234567890";
(account as ClearOpaqueKeysOf<BankAccount>).iban = "TR320010009999901234567890";
(account as ClearOpaqueTokensOf<BankAccount, "BANK">).bank = { code: "TCZBTR2AXXX" };
(account as ClearOpaqueTokensOf<BankAccount, "BANK" | "IBAN">).bank = { code: "TCZBTR2AXXX" };
(account as ClearOpaqueTokensOf<BankAccount, string>).bank = { code: "TCZBTR2AXXX" };
(account as ClearOpaqueTokensOf<BankAccount, unknown>).bank = { code: "TCZBTR2AXXX" };
(account as ClearOpaqueTokensOf<BankAccount, any>).bank = { code: "TCZBTR2AXXX" };
(account as ClearOpaqueTokensOf<BankAccount>).bank = { code: "TCZBTR2AXXX" };
(account as ClearOpaqueTokensOf<BankAccount, "CURRENCY">).currency = { code: "USD", sign: "$" };
(account as ClearOpaqueTokensOf<BankAccount, "CURRENCY" | "IBAN">).currency = { code: "USD", sign: "$" };
(account as ClearOpaqueTokensOf<BankAccount>).currency = { code: "USD", sign: "$" };
(account as ClearOpaqueTokensOf<BankAccount, "IBAN">).iban = "TR320010009999901234567890";
(account as ClearOpaqueTokensOf<BankAccount, "BANK" | "IBAN">).iban = "TR320010009999901234567890";
(account as ClearOpaqueTokensOf<BankAccount>).iban = "TR320010009999901234567890";

// But, these should not be allowed.
(account as ClearOpaqueKeysOf<BankAccount, "iban">).bank = { code: "TCZBTR2AXXX" };
(account as ClearOpaqueKeysOf<BankAccount, "bank">).currency = { code: "USD", sign: "$" };
(account as ClearOpaqueKeysOf<BankAccount, "currency">).iban = "TR320010009999901234567890";
(account as ClearOpaqueTokensOf<BankAccount, "IBAN">).bank = { code: "TCZBTR2AXXX" };
(account as ClearOpaqueTokensOf<BankAccount, "BANK">).currency = { code: "USD", sign: "$" };
(account as ClearOpaqueTokensOf<BankAccount, "CURRENCY">).iban = "TR320010009999901234567890";
```

Please define the `Opaque`, `Transparent`, `OpaqueKeysOf`, `OpaqueTokensOf`, `ClearOpaqueKeysOf`, and `ClearOpaqueTokensOf` types.

<details>
  <summary>Answer</summary>

```typescript
declare const brand: unique symbol;

type Branded<Token extends string> = {
  readonly [brand]: Token;
};

type Opaque<T, Token extends string> = T & Branded<Token>;

type Transparent<T extends Branded<any>> =
  T extends Opaque<infer U, T[typeof brand]>
    ? U
    : T;

type OpaqueKeysOf<T> = {
  [P in keyof T]: T[P] extends Branded<any> ? P : never;
}[keyof T];

type OpaqueTokensOf<T> = {
  [P in keyof T]: T[P] extends Branded<infer U> ? U : never;
}[keyof T];

type ClearOpaqueKeysOf<T, K = OpaqueKeysOf<T>> = {
  [P in keyof T]: P extends K
    ? T[P] extends Branded<any>
      ? Transparent<T[P]>
      : T[P]
    : T[P];
};

type ClearOpaqueTokensOf<T, K = OpaqueTokensOf<T>> = {
  [P in keyof T]: T[P] extends Branded<infer Token>
    ? Token extends K
      ? Transparent<T[P]>
      : T[P]
    : T[P];
};
```

This is an approach to nominal typing in TypeScript, as described in this official example: https://typescriptlang.org/play#example/nominal-typing

</details>

## Q25

```typescript
interface App {
  run(): void;
}

type T1 = TupleOf<boolean, 4>; // [boolean, boolean, boolean, boolean];
type T2 = TupleOf<number, 5>;  // [number, number, number, number, number];
type T3 = TupleOf<Date, 6>;    // [Date, Date, Date, Date, Date, Date];
type T4 = TupleOf<string, 99>; // [string, ... 97 more ..., string];
type T5 = TupleOf<App, 999>;   // [App, ... 997 more ..., App];
```

Please declare the `TupleOf` type. You may hit a limit depending on your TypeScript version (use v4.5+) and implementation.

<details>
  <summary>Answer</summary>

```typescript
type TupleOf<
  T,
  N extends number,
  Acc extends T[] = []
> = N extends Acc['length']
  ? Acc
  : TupleOf<T, N, [...Acc, T]>;
```

We don't have to worry about negative numbers, decimals, or `Infinity` as input because they will hit the limit and TypeScript will give an error.

</details>

## Q26

```typescript
type HelloWorld1 = Concat<["Hello", " ", "world"]>;
// "Hello world"

type HelloWorld2 = Concat<["H", "e", "l", "l", "o", " ", "w", "o", "r", "l", "d"]>;
// "Hello world"

type SingleLetter = Concat<["X"]>;
// "X"

type EmptyString = Concat<[]>;
// ""

type NonString = Concat<["a", "b", 3]>;
//                            ^
// Error: Type 'number' is not assignable to type 'string';
```

Please declare the `Concat` type.

<details>
  <summary>Answer</summary>

```typescript
type Concat<T extends string[]> =
  T extends [
    infer Head extends string,
    ...infer Tail extends string[]
  ]
    ? `${Head}${Concat<Tail>}`
    : "";
```

When the iteration is complete and `Head` becomes undefined, the outer condition fails because `undefined` doesn't extend `string`.

</details>

## Q27

```typescript
type T0 = Split<"">;
// []

type T1 = Split<"X">;
// ["X"]

type T2 = Split<"Hello world">;
// ["H", "e", "l", "l", "o", " ", "w", "o", "r", "l", "d"]

type T3 = Split<"CR#7">;
// ["C", "R", "#", "7"]
```

Please declare the `Split` type.

<details>
  <summary>Answer</summary>

```typescript
type Split<T extends string> =
  T extends `${infer Head}${infer Tail}`
    ? [Head, ...Split<Tail>]
    : [];
```

Adding a delimiter parameter to both `Split` and `Concat` (from [#Q26](#q26)) is possible:

```typescript
type Split<T extends string, Delimiter extends string = ""> =
  "" extends T
    ? []   
    : T extends `${infer Head}${Delimiter}${infer Tail}`
      ? [Head, ...Split<Tail, Delimiter>]
      : [T];

type Concat<T extends string[], Delimiter extends string = ""> = 
  T extends [infer Head extends string, ...infer Tail extends string[]]
    ? `${Head}${Delimiter}${Concat<Tail>}`
    : "";


type S0 = Split<"">;  // []
type C0 = Concat<S0>; // ""

type S1 = Split<"", "X">;  // []
type C1 = Concat<S1, "X">; // ""

type S2 = Split<"X">; // ["X"]
type C2 = Concat<S2>; // "X"

type S3 = Split<"X", "X">; // [""]
type C3 = Concat<S3, "X">; // "X"

type S4 = Split<"Hello world", " ">; // ["Hello", "world"]
type C4 = Concat<S4, " ">;           // "Hello world"

type S5 = Split<"CR#7">; // ["C", "R", "#", "7"]
type C5 = Concat<S5>;    // "CR#7"

type S6 = Split<"CR#7", "#">; // ["CR", "7"]
type C6 = Concat<S6, "#">;    // "CR#7"
```

</details>

## Q28

```typescript
type T0 = Reverse<[]>;               // []
type T1 = Reverse<[boolean]>;        // [boolean]
type T2 = Reverse<[null, void]>;     // [void, null]
type T3 = Reverse<["x", "y", "z"]>;  // ["z", "y", "x"]
type T4 = Reverse<[0, 1, 2, 3, 4]>;  // [4, 3, 2, 1, 0]
type T5 = Reverse<[{}, object]>;     // [object, {}]
```

Please declare the `Reverse` type.

<details>
  <summary>Answer</summary>

```typescript
type Reverse<T extends unknown[]> =
  T extends [infer Head, ...infer Tail]
    ? [...Reverse<Tail>, Head]
    : [];
```

Bonus (string variant):

```typescript
type ReverseStr<T extends string> =
  T extends `${
    infer Head extends string
  }${
    infer Tail extends string
  }`
    ? `${ReverseStr<Tail>}${Head}`
    : '';

type T0 = ReverseStr<''>; // ""
type T1 = ReverseStr<'X'>; // "X"
type T2 = ReverseStr<'hello'>; // "olleh"
```

</details>

## Q29

```typescript
class Bar extends Foo {}
class Baz extends Foo {
  constructor(public readonly x: number, public readonly y: number) {
    super();
  }
}
class Qux extends Baz {}

const foo = Foo.create();
//           ^
// Error: Cannot assign an abstract constructor type to a non-abstract constructor type.

const bar = Bar.create();
// OK. Type of bar is Bar.

const baz = Baz.create();
//                 ^
// Error: Expected 2 arguments, but got 0.

const qux = Qux.create(0, 1);
// OK. Type of qux is Qux.
```

Please declare the abstract `Foo` class and implement the static `create` method. It doesn't have to do anything other than returning an instance.

<details>
  <summary>Answer</summary>

```typescript
abstract class Foo {
  static create<
    T extends Foo,
    Args extends unknown[]
  >(this: new (...args: Args) => T, ...args: Args) {
    return new this(...args);
  }
}
```

Info: https://typescriptlang.org/docs/handbook/2/classes.html#static-members

</details>

## Q30

```typescript
type T01 = IsGTE<0, 0>; // true
type T02 = IsGTE<0, 1>; // false
type T03 = IsGTE<1, 0>; // true
type T04 = IsGTE<1, 1>; // true
type T05 = IsGTE<42, 24>; // true
type T06 = IsGTE<42, 99>; // false
type T07 = IsGTE<999, 1000>; // false
type T08 = IsGTE<1000, 999>; // true
type T09 = IsGTE<1000, 1000>; // true
type T10 = IsGTE<-0, -0>; // true
type T11 = IsGTE<-0, -1>; // true
type T12 = IsGTE<-1, -0>; // false
type T13 = IsGTE<-1, -1>; // true
type T14 = IsGTE<-42, -24>; // false
type T15 = IsGTE<-42, -99>; // true
type T16 = IsGTE<-999, -1000>; // true
type T17 = IsGTE<-1000, -999>; // false
type T18 = IsGTE<-1000, -1000>; // true
type T19 = IsGTE<0, -0>; // true
type T20 = IsGTE<0, -1>; // true
type T21 = IsGTE<1, -1>; // true
type T22 = IsGTE<42, -24>; // true
type T23 = IsGTE<42, -42>; // true
type T24 = IsGTE<1000, -1000>; // true
type T25 = IsGTE<-0, 0>; // true
type T26 = IsGTE<-1, 0>; // false
type T27 = IsGTE<-1, 1>; // false
type T28 = IsGTE<-42, 0>; // false
type T29 = IsGTE<-42, 42>; // false
type T30 = IsGTE<-1000, 1000>; // false
```

Please declare the `IsGTE` type. Generic parameters will always be numeric literals.

<details>
  <summary>Answer</summary>

```typescript
type And<T extends (() => boolean)[], I extends 0[] = []> = 
  I['length'] extends T['length']
    ? true
    : ReturnType<T[I['length']]> extends false
      ? false
      : And<T, [...I, 0]>;

type Or<T extends (() => boolean)[], I extends 0[] = []> =
  I['length'] extends T['length']
    ? false
    : ReturnType<T[I['length']]> extends true
      ? true
      : Or<T, [...I, 0]>;

type Not<T extends boolean> = T extends true ? false : true;

type IsEqual<N1, N2> =
  (<T>() => (T extends N1 ? 1 : 0)) extends
  (<T>() => (T extends N2 ? 1 : 0))
    ? true
    : false;

type IsNegative<T extends number> = `${T}` extends `-${number}` ? true : false;

type IsSmallerNegative<
  A extends string,
  B extends string,
  I extends 0[] = []
>
  = `-${I['length']}` extends infer M extends A | B
    ? M extends A ? false : true
    : IsSmallerNegative<A, B, [...I, 0]>;

type IsGTE<
  A extends number,
  B extends number
> =
  Or<[
    () => IsEqual<A, B>,

    () => And<[
      () => IsNegative<B>,
      () => Not<IsNegative<A>>
    ]>,

    () => And<[
      () => IsNegative<A>,
      () => IsNegative<B>,
      () => IsSmallerNegative<`${B}`, `${A}`>
    ]>,
    
    () => And<[
      () => Not<IsNegative<A>>,
      () => Not<IsNegative<B>>,
      () => IsSmallerNegative<`-${A}`, `-${B}`>
    ]>,
  ]>;
```

</details>
