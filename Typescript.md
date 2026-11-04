# TypeScript Basics

## Contents

1. [Foundations](#1-foundations) — functions, inference, annotation, literal types
2. [Arrays, Tuples & Enums](#2-arrays-tuples--enums)
3. [Union Types & Special Types](#3-union-types--special-types) — narrowing, `unknown`, `never`
4. [Objects, Aliases & Interfaces](#4-objects-aliases--interfaces) — extends, merging, interface patterns, structural typing
5. [Generics](#5-generics) — constraints, multiple parameters, async, type assertion
6. [Utility Types](#6-utility-types) — `Partial`, `Pick`, `Omit`, `Required`
7. [Classes & Object-Oriented TypeScript](#7-classes--object-oriented-typescript) — access modifiers, getters/setters, abstract classes

---

## 1. Foundations

### Variables & Functions

```ts
const firstNumber = 5;

function calculateSum(a: number, b: number) {
    return a + b;
};

const result = calculateSum(firstNumber, 10);
console.log(`Result: ${result}`);
```

### Function Parameter Type Checking

```ts
function printData(data: string): void {
    console.log(data);
}

printData("Hello, World!");
// printData(9); // This will cause a TypeScript error because 9 is not a string
```

### Type Inference

TypeScript infers the type from the assigned value, so annotations are often unnecessary.

```ts
const numbers = [1, 2, 3, 4, 5];
const userName = 'John Doe';
```

### Type Annotation

Annotate explicitly when the inferred type is too broad, or when you want a narrower type.

```ts
let drink = "coffee";
// drink = 90;

let age: number = 30;
// age = "thirty";
```

### Literal Types

A type can be restricted to a specific set of allowed values.

```ts
let paymentStatus: "paid" | "unpaid" | "pending" = "paid";

paymentStatus = "unpaid";
// paymentStatus = "completed"; // This will cause a TypeScript error because "completed" is not a valid value for paymentStatus
```

## 2. Arrays, Tuples & Enums

### Arrays

```ts
function getTotal(numbers: Array<number>) {
    return numbers.reduce((acc, curr) => acc + curr, 0);
}

console.log(getTotal([1, 2, 3, 4, 5]));
```

```ts
const chaiArray: string[] = ["Masala Chai", "Green Tea", "Black Tea"];

chaiArray.push("Herbal Tea"); // valid, the array is mutable
```

### Readonly Arrays

```ts
const readOnlyChaiArray: readonly string[] = ["Masala Chai", "Green Tea", "Black Tea"];
readOnlyChaiArray.push("Herbal Tea"); // This will cause a TypeScript error because the array is read-only
```

### Tuples

Tuples are fixed-length arrays where each position has a known type. Elements can also be optional.

```ts
let chaiTuple: [string, number, boolean?];
chaiTuple = ["Masala Chai", 5.99, true]; // valid
chaiTuple = ["Green Tea", 3.99]; // valid
// chaiTuple = [5.99, "Masala Chai"]; // This will cause a TypeScript error because the types are in the wrong order
```

### Enums

Numeric enums are auto-numbered from `0`:

```ts
enum PaymentStatus {
    Paid,
    Unpaid,
    Pending
}

console.log(PaymentStatus.Paid); // Output: 0
```

String enums require an explicit value for each member:

```ts
enum ChaiType {
    Masala = "masala",
    Green = "green",
    Black = "black",
    Herbal = "herbal"
}

function getChaiType(chai: ChaiType): void {
    console.log(`The chai type is: ${chai}`);
}

getChaiType(ChaiType.Herbal); // Output: The chai type is: herbal
// getChaiType("herbal"); // This will cause a TypeScript error because a raw string is not assignable to a string enum
```

## 3. Union Types & Special Types

### Union Types

A value may be one of several types. `typeof` narrowing tells the compiler which branch is active.

```ts
type StringOrNumber = string | number;

function displayValue(value: StringOrNumber) {
    if (typeof value === 'string') {
        console.log(`String value: ${value}.toUpperCase() => ${value.toUpperCase()}`);
    } else {
        console.log(`Number value: ${value}.toFixed(2) => ${value.toFixed(2)}`);
    }
}

displayValue('Hello');
displayValue(42);
```

```ts
function getFirstThree(data: string | number[]) {
    return data.slice(0, 3);
}

console.log(getFirstThree('Hello World')); // Returns 'Hel'
console.log(getFirstThree([1, 2, 3, 4, 5])); // Returns [1, 2, 3]
```

### Unknown Type

`unknown` is the type-safe counterpart of `any` — you must narrow it before using it.

```ts
let newValue: unknown;

newValue = "Hello";
newValue = 42;

if (typeof newValue === "string") {
    console.log(newValue.toUpperCase());
}
```

### Never Type

`never` represents a value that can never occur. After exhausting every union member, the remaining value is `never`.

```ts
type Role = "admin" | "user";

function checkRole(role: Role): void {
    if (role === "admin") {
        console.log("You have admin access.");
        return;
    }
    if (role === "user") {
        console.log("You have user access.");
        return;
    }
    role;
}
```

## 4. Objects, Aliases & Interfaces

### Type Aliases

```ts
type User = {
    name: string;
    age: number;
    address?: string;
}

const user: User = {
    name: 'John Doe',
    age: 30
};
```

### Interfaces

```ts
interface Transaction {
    payer: string;
    payee: string;
};

interface BankAccount {
    accountNumber: string;
    balance: number;
    transactions: Transaction[];
};

const transaction1: Transaction = {
    payer: 'Alice',
    payee: 'Bob'
};

const transaction2: Transaction = {
    payer: 'Charlie',
    payee: 'David'
};

const account: BankAccount = {
    accountNumber: '123456789',
    balance: 1000,
    transactions: [transaction1, transaction2]
};
```

### Interface Extends

```ts
interface Person {
    name: string;
    age: number;
};

interface Employee extends Person {
    employeeId: string;
    department: string;
};

const employee: Employee = {
    name: 'Jane Smith',
    age: 28,
    employeeId: 'E12345',
    department: 'Engineering'
};
```

### Interface Merging (Declaration Merging)

Declaring the same interface twice merges the members into one type.

```ts
interface UserProfile {
    username: string;
    email: string;
};

interface UserProfile {
    age: number;
    address?: string;
};

const userProfile: UserProfile = {
    username: 'johndoe',
    email: 'john.doe@example.com',
    age: 30,
    // address: '123 Main St'
};
```

### Interface Patterns

#### Call Signatures

An interface can describe a **callable** value (a function type), rather than an object's properties:

```ts
interface DiscountCalculator {
    (price: number): number;
}

const calculator: DiscountCalculator = (p) => p * 0.2;
calculator(3);

// const badCalculator: DiscountCalculator = (p: string) => p; // This will cause a TypeScript error because the parameter type does not match
```

#### Index Signatures

Index signatures describe objects with arbitrary keys whose values all share one type:

```ts
interface ChaiRating {
    [flavour: string]: number;
}

const ratings: ChaiRating = {
    masala: 4.5,
    ginger: 5
};

// ratings.elaichi = "5"; // This will cause a TypeScript error because values must be numbers
```

#### Readonly Properties

Individual interface properties can be marked `readonly`, so they can only be set when the object is created:

```ts
interface ChaiItem {
    readonly name: string;
    price: number;
}

const modernChai: ChaiItem = {
    name: "Erani",
    price: 45
};

modernChai.price = 50; // valid
// modernChai.name = "Masala"; // This will cause a TypeScript error because name is readonly
```

### Structural Typing / Duck Typing

TypeScript compares shapes, not names — an object with at least the required members is assignable.

```ts
interface Animal {
    name: string;
    category: string;
}

function logAnimal(animal: Animal): boolean {
    console.log(`Animal Name: ${animal.name}, Category: ${animal.category}`);
    return true;
}

const tiger = {
    name: 'Tiger',
    category: 'Mammal',
    habitat: 'Forest'
}

logAnimal(tiger);
```

```ts
interface Vehicle {
    make: string;
    model: string;
    year: number;
    sound(model: string, year: number): void;
}

const car: Vehicle = {
    make: 'Toyota',
    model: 'Camry',
    year: 2020,
    sound(model: string, year: number) {
        console.log(`The ${year} ${model} is making a sound!`);
    }
}
```

## 5. Generics

### `any` vs Generic

`any` switches type checking off entirely:

```ts
function logData(data: any) {
    console.log(`Data: ${data} ${Object.prototype.toString.call(data)}`);
}

logData('Hello, World!'); // Logs: Data: Hello, World! [object String]
logData(42); // Logs: Data: 42 [object Number]
logData([1, 2, 3]); // Logs: Data: 1,2,3 [object Array]
```

A generic preserves the incoming type instead of erasing it:

```ts
function logData<T>(data: T): T {
    console.log(`Data: ${data} ${Object.prototype.toString.call(data)}`);
    return data;
}

logData({ name: 'John Doe', age: 30 }); // Logs: Data: [object Object] [object Object]
```

### Multiple Type Parameters

A generic can take more than one type parameter. Each is inferred independently, which is handy for pairing unrelated values:

```ts
function pair<A, B>(a: A, b: B): [A, B] {
    return [a, b];
}

pair("Elaichi Chai", { price: 30 }); // A = string, B = { price: number }
pair(3, 5); // A = number, B = number

// Type parameters can also be passed explicitly
pair<string, number>("Elaichi Chai", 30);
```

### Generic Constraints

`extends` restricts which types a generic accepts — here, anything with an `age` number.

```ts
interface HasAge {
    age: number;
}

const playersWithAge: HasAge[] = [
    { age: 30 },
    { age: 25 },
    { age: 35 }
];

function getYoungest<T extends HasAge>(people: T[]): T | undefined {
    if (people.length === 0) return undefined;
    return [...people].sort((a, b) => a.age - b.age)[0];
};

getYoungest(playersWithAge)?.age;
```

```ts
interface Player {
    name: string;
    age: number;
    old: boolean;
}

const player: Player[] = [
    { name: 'Alice', age: 25, old: false },
    { name: 'Bob', age: 30, old: false },
    { name: 'Charlie', age: 35, old: true }
];
getYoungest(player)?.name;
```

### Type Assertion

`as` tells the compiler to treat a value as a specific type. Prefer narrowing when possible.

```ts
const person = getYoungest(player) as Player;
person.name;
```

### Generic Async Functions

```ts
interface User {
    name: string;
    age: number;
}

interface Posts {
    title: string;
    content: string;
}

const getUserData = async (path: string): Promise<User> => {
    const response = await fetch(`https://jsonuserdata${path}`);
    return response.json();
}

const getPostsData = async <ResultType>(path: string): Promise<ResultType> => {
    const response = await fetch(`https://jsonposts${path}`);
    return response.json();
}

(async () => {
    const userData = await getUserData('/user/1');
    console.log(userData);
})();

(async () => {
    const postsData = await getPostsData<Posts>('/posts/1');
    postsData.content;
})();
```

## 6. Utility Types

Utility types transform an existing type into a new one.

### Partial

```ts
type Chai = {
    name: string;
    price: number;
    isHot: boolean;
}

const updateChai = (updates: Partial<Chai>) => {
    console.log("Updating chai with the following properties:", updates);
}

updateChai({ price: 5.99, isHot: true });
updateChai({ name: "Masala Chai" });
updateChai({}); // This is valid because all properties are optional in Partial<Chai>
```

### Pick

```ts
type ChaiWithOptionalProperties = Pick<Chai, "name" | "price">;

const chaiInfo: ChaiWithOptionalProperties = {
    name: "Green Tea",
    price: 30
}
```

### Omit

```ts
type OmitChai = Omit<Chai, "isHot">;

const chaiWithoutIsHot: OmitChai = {
    name: "Black Tea",
    price: 20
}
```

### Required

```ts
type newChai = {
    name?: string;
    price?: number;
}

const makeChai = (chai: Required<newChai>) => {
    console.log("Making chai with the following properties:", chai);
}

// makeChai({}); // This will cause a TypeScript error because all properties are required in Required<newChai>
makeChai({ name: "Masala Chai", price: 5.99 }); // This is valid because all properties are provided
```

## 7. Classes & Object-Oriented TypeScript

### Class Basics

```ts
class Maggie {
    flavor: string;
    price: number;

    constructor(flavour: string, price: number) {
        this.flavor = flavour;
        this.price = price;
    }

    details() {
        console.log(`Maggie flavor: ${this.flavor}, Price: ${this.price}`);
    }
}

const maggie = new Maggie("Cheese", 4.99);
maggie.details(); // Output: Maggie flavor: Cheese, Price: 4.99
```

### Access Modifiers

- `public` (default) — accessible anywhere
- `private` — accessible only inside the class
- `protected` — accessible inside the class and its subclasses

```ts
class Wallet {
    public name: string = "Raj";

    private balance: number = 1000;

    getBalance() {
        return this.balance;
    }
}

const w = new Wallet();
w.getBalance(); // valid
// w.balance; // This will cause a TypeScript error because balance is private

class Shop {
    protected shopName: string = "Chai Darbaar";
}

class Branch extends Shop {
    getName() {
        return this.shopName; // valid, protected members are visible to subclasses
    }
}

new Branch().getName();
```

### ES Private Fields (`#`)

Native JavaScript private fields are inaccessible even from outside the class at runtime:

```ts
class Book {
    #type: string = "Ramayan";

    getBook() {
        return this.#type;
    }
}

const book = new Book();
book.getBook();
// book.#type; // This will cause a TypeScript error because #type is not accessible outside the class
```

### Getters & Setters

```ts
class ModernChai {
    private _sugar: number = 5;

    get sugar() {
        return this._sugar;
    }

    set sugar(value: number) {
        if (value > 5) throw new Error("Too sweet");
        this._sugar = value;
    }
}

const c = new ModernChai();
c.sugar = 3; // valid
// c.sugar = 8; // Throws: Too sweet
```

### Abstract Classes

Abstract classes cannot be instantiated directly and force subclasses to implement their abstract members:

```ts
abstract class Drink {
    abstract make(): void;
}

class MyChai extends Drink {
    make() {
        console.log("this is abstracted");
    }
}

// new Drink(); // This will cause a TypeScript error because abstract classes cannot be instantiated
new MyChai().make(); // Output: this is abstracted
```
