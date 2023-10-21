+++
title = "Typescript Comprehensive Cheatsheet"
date = "2021-08-14T22:06:15+08:00"
categories = [ "编程",]
tags = [ "typescript",]
toc = "true"
+++


一份详尽的 ts 语法笔记。这周在看组里前端同事的代码，感觉完全还是在写 JS，以我有限的 JS/TS 知识，也知道可以写得更加规范一点。但是一上手开始改，还真是手生。
又重新过了一遍文档，做了一点笔记。

<!--more-->

### install

```bash
# Install
npm install typescript
# Run
npx tsc
# Run with a specific config  
npx tsc --project configs/my_tsconfig.json
# Triple slash directives
# Reference built-in types  
/// <reference lib="es2016.array.include" />
# Reference other types
/// <reference path="../my_types" />
/// <reference types="jquery" />
# AMD
/// <amd-module name="Name" />
/// <amd-dependency path="app/foo" name="foo" />

# Compiler comments
# Don’t check this file
// @ts-nocheck
# Check this file (JS)  
// @ts-check
# Ignore the next line  
// @ts-ignore
# Expect an error on the next line  
// @ts-expect-error

# ignore ts type error
tsc --noEmitOnError hello.ts

# tsconfig.json
"strict": true
"noImplicitAny"
"strictNullChecks"

```

### ts syntax cheatsheet

```typescript

//-------------------------------- Basic type 
any // untyped, use as js, disables all further type checking
unknown // The unknown type represents any value. This is similar to the any type, but is safer because it’s not legal to do anything with an unknown value:
void // void is not the same as undefined.
// a contextual function type with a void return type (type vf = () => void), when implemented, can return any other value, but it will be ignored.

null // prefer use undefined
undefined
// check optional param? if undefined before use it 

never  // unreachable see: https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking

object
// object is not Object. Always use object! object also diff from {}
// In compile time {} doesn't have Object's members and Object has more strict behavior 

boolean
number
string
// The type names String , Number , and Boolean (starting with capital letters) are legal, but refer to
// some special built-in types that will very rarely appear in your code. Always use string , number , or
// boolean for types.

bigint //ES2020
// Creating a bigint via the BigInt function
const oneHundred: bigint = BigInt(100);
// Creating a BigInt via the literal syntax
const anotherHundred: bigint = 100n;

string[] // or  Array<string>
// ReadonlyArray
// error: new ReadonlyArray("red", "green", "blue");
// use:
const roArray: ReadonlyArray<string> = ["red", "green", "blue"];
// Just as TypeScript provides a shorthand syntax for Array<Type> with Type[], 
// it also provides a shorthand syntax for ReadonlyArray<Type> with readonly Type[]
let x: readonly string[] = [];
let y: string[] = [];

[string, number]  // tuple

string | null | undefined   // union type
// use isArray function narrowing the array type
function welcomePeople(x: string[] | string) {
  if (Array.isArray(x)) {
    console.log("Hello, " + x.join(" and "));
  } else {
    console.log("Welcome lone traveler " + x);
  }
}
// use in operator narrowing method structual type
// instanceof narrowing

type direction= 'left' | 'right'; // Literal types
type roll= 1 | 2 | 3 | 4 | 5 | 6;


// ?? (nullish coalescing)  
function getValue(val?: number): number | 'nil' {
  // Will only return 'nil' if `val` is null or undefined
  return val ?? 'nil';
}

// ?. (optional chaining)
function countCaps(value?: string) {
  // The `value` expression be undefined if `value` is null or
  // undefined, or if the `match` call doesn't find anything.
  return value?.match(/[A-Z]/g)?.length ?? 0;
}

// ! (null assertion) skip the null/undefined check
let value: string | undefined;
// ... Code that we're sure will initialize `value` ...
// Assert that `value` is defined
console.log(`value is ${value!.length} characters long`);

// &&=  assign a value only if current value is truthy
let a;
let b = 1;
a &&= 'default'; // a is still undefined
b &&=  5; // b is now 5

// ||=   assign a value only if current value is falsy
let a;
let b = 1;
a ||= 'default'; // a is 'default' now
b ||=  5; // b is still 1

// ??=  assign a value only if current value is null or undefined
let a;
let b = 0;
a ??= 'default'; // a is now 'default'
b ??=  5; // b is still 0

// Object
{
    requiredStringVal: string;
    optionalNum?: number;
    readonly readOnlyBool: bool;

}
// Index signature: object with arbitrary string properties (like a hashmap or dictionary)
{ [key: string]: Type; }
{ [key: number]: Type; }
{ [key: symbol]: Type; }
{ [key: `data-${string}`]: Type; }

// Array of functions that return strings
(() => string)[]
// or
{ (): string; }[]
// or
Array<() => string>

// Basic tuples
let myTuple: [ string, number, boolean? ];
myTuple = [ 'test', 42 ];
// Variadic tuples  
type Numbers = [number, number];
type Strings = [string, string];
 
type NumbersAndStrings = [...Numbers, ...Strings];
// [number, number, string, string]

type NumberAndRest = [number, ...string[]];
// [number, varying number of string]

type RestAndBoolean = [...any[], boolean];
// [varying number of any, boolean]

// Named tuples
type Vector2D = [x: number, y: number];
function createVector2d(...args: Vector2D) {}
// function createVector2d(x: number, y: number): void
// !!!cation, you can not use v2d.x the name is just for hint, not for compiler


// Interface  
interface Child extends Parent, SomeClass {
    property: Type;
    optionalProp?: Type;
    optionalMethod?(arg1: Type): ReturnType;
}
// Class  
class Child extends Parent

implements Child, OtherChild {
    property: Type;
    defaultProperty = 'default value';
    private _privateProperty: Type;
    private readonly _privateReadonlyProperty: Type;
    static staticProperty: Type;

    static {
        try {
            Child.staticProperty = calcStaticProp();
        } catch {
            Child.staticProperty = defaultValue;
        }
    }

    constructor(arg1: Type) {
        super(arg1);
    }

    private _privateMethod(): Type {}

    methodProperty: (arg1: Type) => ReturnType;
    overloadedMethod(arg1: Type): ReturnType;
    overloadedMethod(arg1: OtherType): ReturnType;
    overloadedMethod(arg1: CommonT): CommonReturnT {}
    static staticMethod(): ReturnType {}
    subclassedMethod(arg1: Type): ReturnType {
        super.subclassedMethod(arg1);
    }
}
//-------------------------------- Function
// Function type  
(arg1: Type, argN: Type) => Type;
// or
{ (arg1: Type, argN: Type): Type; }
// Function type with optional param  
(arg1: Type, optional?: Type) => ReturnType
// Function type with rest param  
(arg1: Type, ...allOtherArgs: Type[]) => ReturnType
// Function type with static property
{ (): Type; staticProp: Type; }
// Default argument
function fn(arg1 = 'default'): ReturnType {}
// Arrow function
(arg1: Type): ReturnType => { ...; return value; }
// or
(arg1: Type): ReturnType => value;
// this typing  
function fn(this: Foo, arg1: string) {}
// Overloads  
function conv(a: string): number;
function conv(a: number): string;
function conv(a: string | number): string | number {
    ...
}
// Call Signatures
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}
// Construct Signatures
type SomeConstructor = {
  new (s: string): SomeObject;
};
function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
// Generic Functions
function firstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0];
}
// Constraints
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}
 
// longerArray is of type 'number[]'
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString is of type 'alice' | 'bob'
const longerString = longest("alice", "bob");
// Error! Numbers don't have a 'length' property
const notOK = longest(10, 100);

// Working with Constrained Values
// 期望返回 Type，而不是具有{ length: number }约束的类型。即期望子类，返回了父类，会导致属性变少
function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number
): Type {
  if (obj.length >= minimum) {
    return obj;
  } else {
    return { length: minimum }; // Type '{ length: number; }' is not assignable to type 'Type'.
    // '{ length: number; }' is assignable to the constraint of type 'Type', but 'Type' could be instantiated with a different subtype of constraint '{ length: number; }'
  }
}
// Rule: If a type parameter only appears in one location, strongly reconsider if you actually need it
// Rule: Always use as few type parameters as possible
// Rule: When possible, use the type parameter itself rather than constraining it

// 函数重载
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
// 上面两个为函数重载签名
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3);
// Always prefer parameters with union types instead of overloads when possible


//-------------------------------- Enum
// Unlike most TypeScript features, this is not a type-level addition to JavaScript but something added to the language and runtime.
// Because of this, it’s a feature which you should know exists, but maybe hold off on using unless you are sure
enum Color {Red, Green, Blue = 4} // default get Red=0, all of the following members are auto-incremented Green=1, except you give.
let c: Color = Color.Green
// enum 有两种，value 是 number（默认）或 string 的。
// !!!numeric enums members also get a reverse mapping from enum values to enum names, for example:
// !!! careful with numeric enum iteration
enum LogLevel {
  ERROR,
  WARN,
  INFO
}
for (let element in LogLevel) {
        // 先遍历 index，再遍历 value
        console.log(element +" - "+ LogLevel[element]);// output
        // [LOG]: "0 - ERROR"
        // [LOG]: "1 - WARN"
        // [LOG]: "2 - INFO"
        // [LOG]: "ERROR - 0"
        // [LOG]: "WARN - 1"
        // [LOG]: "INFO - 2"
}
// string value enum 没有上面这个问题。
// 在编译内部，ts 编译得到一个 name->value value->name 的双向 map
enum Enum {
  VAL,
}

// tsc output
"use strict";
var Enum;
(function (Enum) {
    Enum[Enum["VAL"] = 0] = "VAL";
})(Enum || (Enum = {}));


// for in 是 ES5 标准，遍历 key.  in 会遍历原型链 prototype 上面的属性。非底层代码，禁止使用 for..in.
// for of 是 ES6 标准，遍历 value.

// 语法上 enum 允许 string value 和 numeric value 并存，但是代码业务含义
enum BooleanLikeHeterogeneousEnum {
  No = 0,
  Yes = "YES",
}

// Try to use ts.forEach, ts.map, and ts.filter instead of loops when it is not strongly inconvenient.


//-------------------------------- Type alias

type Name = string;
type Direction = 'left' | 'right';
type ElementCreator = (type: string) => Element;
type Point = { x: number, y: number };
type Point3D = Point & { z: number };
type PointProp = keyof Point; // 'x' | 'y'
const point: Point = { x: 1, y: 2 };
type PtValProp = keyof typeof point; // 'x' | 'y'
// Extending a type via intersections
type Animal = {
  name: string
}

type Bear = Animal & { 
  honey: boolean 
}

// type alias , interface 区别



//-------------------------------- Generics
// Function using type parameters
<T>(items: T[], callback: (item: T) => T): T[]
// Interface with multiple types  
interface Pair<T1, T2> {
    first: T1;
    second: T2;
}
// Constrained type parameter
<T extends ConstrainedType>(): T
// Default type parameter  
<T = DefaultType>(): T
// Constrained and default type parameter
<T extends ConstrainedType = DefaultType>(): T
// Generic tuples
type Arr = readonly any[];
 
function concat<U extends Arr, V extends Arr>(a: U, b: V):
[...U, ...V] { return [...a, ...b] }
 
const strictResult = concat([1, 2] as const, ['3', '4'] as const);
const relaxedResult = concat([1, 2], ['3', '4']);
 
// strictResult is of type [1, 2, '3', '4']
// relaxedResult is of type (string | number)[]

//-------------------------------- Index, mapped, and conditional types
// Index type query (keyof)
type Point = { x: number, y: number };
let pointProp: keyof Point = 'x';

function getProp<T, K extends keyof T>(
    val: T,
    propName: K

): T[K] { ... }

// Mapped types
// see more: https://www.typescriptlang.org/docs/handbook/utility-types.html
type Stringify<T> = { [P in keyof T]: string; }
type Partial<T> = { [P in keyof T]?: T[P]; }
// Conditional types  
type Swapper = <T extends number | string>
    (value: T) => T extends number ? string : number;
// is equivalent to
(value: number) => string // if T is number, or
(value: string) => number // if T is string

// Conditional mapped types
interface Person {
    firstName: string;
    lastName: string;
    age: number;
}

type StringProps<T> = {
    [K in keyof T]: T[K] extends string ? K : never;
};

type PersonStrings = StringProps<Person>;
// PersonStrings is "firstName" | "lastName"


// Utility types
// Partial  
Partial<{ x: number; y: number; z: number; }>
// is equivalent to
{ x?: number; y?: number; z?: number; }

// Readonly
Readonly<{ x: number; y: number; z: number; }>
// is equivalent to
{
    readonly x: number;
    readonly y: number;
    readonly z: number;
}

//-------------------------------- Pick
Pick<{ x: number; y: number; z: number; }, 'x' | 'y'>
// is equivalent to
{ x: number; y: number; }

// Record, 和 index signature 区别是 后者 key 限制在 string number symbol
Record<'x' | 'y' | 'z', number>
// is equivalent to
{ x: number; y: number; z: number; }

// Exclude  
type Excluded = Exclude<string | number, string>;
// is equivalent to
number

// Extract  
type Extracted = Extract<string | number, string>;
// is equivalent to
string

// NonNullable  
type NonNull = NonNullable<string | number | void>;
// is equivalent to
string | number

// ReturnType
type ReturnValue = ReturnType<() => string>;
// is equivalent to
string

// InstanceType
class Renderer() {}
type Instance = InstanceType<typeof Renderer>;
// is equivalent to
Renderer


// Type guards
// Type predicates  
function isThing(val: unknown): val is Thing {
    // return true if val is a Thing
}
if (isThing(value)) {
    // value is of type Thing
}

//-------------------------------- typeof
// "string"
// "number"
// "bigint"
// "boolean"
// "symbol"
// "undefined"
// "object"
// "function"
declare value: string | number | boolean;
const isBoolean = typeof value === "boolean";
if (typeof value === "number") {
    // value is of type Number
} else if (isBoolean) {
    // value is of type Boolean
} else {
    // value is a string
}

// False
// 0
// NaN
// "" (the empty string)
// 0n (the bigint version of zero)
// null
// undefined

// instanceof
declare value: Date | Error | MyClass;
const isMyClass = value instanceof MyClass;

if (value instanceof Date) {
    // value is a Date
} else if (isMyClass) {
    // value is an instance of MyClass
} else {
    // value is an Error
}

// in 属性和方法
interface Dog { woof(): void; }
interface Cat { meow(): void; }

function speak(pet: Dog | Cat) {
    if ('woof' in pet) {
        pet.woof()
    } else {
        pet.meow()
    }
}


//-------------------------------- Assertions
// Type
let val = someValue as string;
// or
let val = <string>someValue;
Const (immutable value)
let point = { x: 20, y: 30 } as const;
// or
let point = <const>{ x: 20, y: 30 };

function handle(url:string,method: "GET"|"POST"){
    console.log("handle")
}

const req = { url: "https://example.com", method: "GET" };
handle(req.url, req.method as "GET"); 
// remove as "GET", throw: Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.(2345)
// or
const req = { url: "https://example.com", method: "GET" as "GET" };
// or convert to type literals
const req = { url: "https://example.com", method: "GET" } as const;

//-------------------------------- Indexed Access Types
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"]; // number
type I1 = Person["age" | "name"]; // type I1 = string | number
type I2 = Person[keyof Person]; // type I2 = string | number | boolean
type AliveOrName = "alive" | "name";
type I3 = Person[AliveOrName]; // type I3 = string | boolean
     

//-------------------------------- Ambient declarations
// Global
declare const $: JQueryStatic;
// Module
declare module "foo" {
    export class Bar { ... }
}
// Wildcard module  
declare module "text!*" {
    const value: string;
    export default value;
}

```