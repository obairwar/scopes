# Object Oriented Programming in JavaScript

---

## Table of Contents
1. [Classes](#classes)
2. [Constructor](#constructor)
3. [Objects & the `new` keyword](#objects--the-new-keyword)
4. [The `this` keyword](#the-this-keyword)
5. [Encapsulation Problem — Why Private Fields?](#encapsulation-problem--why-private-fields)
6. [Problem with `get`/`set` Before Private Fields](#problem-with-getset-before-private-fields)
7. [Private Fields with `#`](#private-fields-with-)
8. [Getters & Setters (After Private Fields)](#getters--setters-after-private-fields)
9. [Intermediate Version — Mix of Private & Public Fields](#intermediate-version--mix-of-private--public-fields)
10. [Complete Product Class (Full Evolved Version)](#complete-product-class-full-evolved-version)
11. [Static Keyword](#static-keyword)
12. [Function Constructors (Pre-ES6)](#function-constructors-pre-es6)
13. [Builder Design Pattern](#builder-design-pattern)
14. [Object Destructuring](#object-destructuring)
15. [Array Destructuring](#array-destructuring)
16. [Spread Operator & Rest Parameter](#spread-operator--rest-parameter)
17. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

## Classes

In software engineering, anything we encounter can be mapped to a blueprint or a vision. For example, on Amazon every product has a similar structure — we can generalise this and make sure we don't repeat anything again and again.

> **Classes are blueprints for a set of real-life entities.**

They define:
- **Properties** (data members) — what the entity *looks like* (features it possesses)
- **Behaviours** (member functions) — what *actions* can be performed on it

If two objects of the same class are different from each other, then at least one of their data members must have a different value.

### Example: Product Class Properties (Data Members)
- `name` — Name of the product
- `price` — Price of the product
- `company` — Company the product belongs to
- `reviews` — Set of feedbacks/reviews
- `rating` — Rating of the product
- `description` — Description of the product

### Example: Product Class Behaviours (Member Functions)
- `addToCart()`
- `removeFromCart()`
- `displayProduct()`
- `wishlistProduct()`
- `rateProduct()`
- `addReview()`
- `buyProduct()`

### Syntax

```js
class NameOfTheClass {
    // data members and member functions go here
}
```

### Basic Product Class (Before Encapsulation)

```js
class Product {
    name;
    price;
    category;
    description;
    rating;

    addToCart() {
        console.log("Product added to cart");
    }

    removeFromCart() {
        console.log("Product removed from cart");
    }

    displayProduct() {
        console.log("Product displayed");
    }

    buyProduct() {
        console.log("Product bought");
    }
}
```

---

## Constructor

Every class in JS has one special member function called the **constructor**. What is so special about it?

Whenever we create an object using a class, the **constructor is the first method that is automatically called by JS**. It is already available for any class you make — the default version is called the **default constructor**.

We can change the implementation of the default constructor by writing our own:

```js
class Product {
    constructor() {
        // This is your custom constructor
    }
}
```

### Constructor with Parameters

```js
class Product {
    constructor(productName, productPrice, productCategory, productDescription, productRating) {
        this.name = productName;
        this.price = productPrice;
        this.category = productCategory;
        this.description = productDescription;
        this.rating = productRating;
    }
}
```

---

## Objects & the `new` keyword

Using classes, the final entity we develop is called an **object**. Objects are concrete instances of a class.

There is a keyword in JS called `new` which helps us create an object out of a class.

```js
let iphone = new Product("Iphone 11", 900, "Electronics", "Apple Iphone 11", 4.5);
```

> Don't map the use of `new` in JS with other languages — it works very differently.

### How `new` works — 4 Steps

1. Creates a **brand new, plain, and absolutely empty object**.
2. Calls the **constructor** of the class and passes the newly created object (not as a parameter) but inside the `this` keyword. So the constructor automatically has access to `this`, and `this` refers to the plain object from step 1. The constructor logic then executes.
3. `new` triggers everything needed for **prototypes** to work (discussed separately).
4. **Returns the object:**
   - If the constructor **manually returns an object** → that manually returned object is stored in the called variable.
   - If **nothing is returned**, or a **non-object value** (like a number, string) is returned → the constructor doesn't care and automatically returns the value inside `this`.

```
                    Are we returning an object manually from constructor?
                                        │
                    ┌───────────────────┴───────────────────┐
                   YES                                       NO
                    │                                        │
    Then this manual object only              Did we return anything else apart from object?
    will be given back to the caller                        │
                    │                          ┌────────────┴────────────┐
    constructor() {                           YES                        NO (didn't return anything)
        return {x: 10}                         │                         │
    }                               constructor() {          constructor() {
                                        return 10;               // nothing returned
                                    }                        }
                                                │                         │
                                                └────────────┬────────────┘
                                                             │
                                            constructor will automatically
                                            return the value of this keyword
```

> **Important:** `return {}` from a constructor returns an **empty plain object** (not `this`), because `{}` is an object. This is used intentionally when we want to signal that object creation failed — the caller gets a useless empty object instead of a partially-constructed one. This is exactly what the Builder pattern uses when price validation fails.

---

## The `this` keyword

- `this` in JS works very differently than other languages.
- `this` is available to be accessed in any function, outside any function, and in classes as well.

> **In most cases, `this` refers to the call site.**

The **call site** is the entity which is calling/triggering the function — it can be an object, a position, or even the `new` keyword.

```js
let obj = {
    x: 1,
    y: 2,
    fn: function() {
        console.log(this.x, this.y); // this → obj (obj is the call site)
    }
}
obj.fn(); // Output: 1 2
// obj.fn() — obj is the call site because obj is calling fn which has this
```

### Local variables vs `this` — they are NOT the same

A common confusion: local variables inside a function and properties on `this` are completely separate things.

```js
let obj = {
    x: 1,
    y: 2,
    fn: function() { 
        let x = 3;  // local variable x — separate from this.x
        let y = 4;  // local variable y — separate from this.y

        const printVariables = () => {    // arrow function 
            console.log(this.x, this.y); // prints 1 2 — from the object (via lexical this)
            console.log(x, y);           // prints 3 4 — local variables
        }
        printVariables();
    }
}
obj.fn();
// Output:
// 1 2
// 3 4
```

`this.x` and `this.y` refer to the object's properties. `x` and `y` (without `this`) refer to local variables in scope. They are independent.

## The Behavior Across Function Types:
When you nest a function inside an object's method, the choice of function syntax completely changes how this is resolved, while plain variables behave identically.

# Case A: Inside an Arrow Function (() => {})
`this.x` and `this.y` --> Inherited Context: Arrow functions do not have their own `this`. They grab `this` lexically from the parent function (fn). Since `obj.fn()` was invoked, the parent's` this` is `obj`. Thus, `this.x` finds `obj.x `(1).

# Case B: Inside a Regular Function (function() {})
`this.x` and `this.y` --> Broken/Global Context: Regular functions define` this `dynamically based on how they are called. Because the inner function is invoked standalone =>(directly by its name without an object context)=(printVariables()), its `this` falls back to the global object (or undefined in strict mode). It cannot see `obj.x`, resulting in undefined.x and y 

### Exception: Arrow Functions

`this` keyword will **not refer to the call site** if used inside an **arrow function**.

In case of arrow functions, `this` is resolved using **lexical scoping** — it looks outward to the enclosing scope to find `this`.

```js
let obj = {
    x: 10,
    y: 20,
    fn: function() {
        const arrow = () => {
            console.log(this.x, this.y); // this resolved lexically → obj
        }
        arrow();
    }
}
obj.fn(); // Output: 10 20
```

#### How lexical resolution works step by step:
1. Is `this` defined in the scope of the arrow function? **No**
2. Go one scope up — inside function `fn`
3. Is `this` defined inside `fn`? **Yes** — because `fn` is a normal function, we have a definition of `this` inside it (its call site)
4. Who is the call site? `obj` — because `obj` is responsible for triggering `fn`
5. Hence `this` refers to `obj` → output: `10 20`

> When we make a new object using the `new` keyword, the plain object created is the call site for the constructor — so `this` keyword refers to that plain object altogether.

---

## Encapsulation Problem — Why Private Fields?

In the current basic implementation, we don't add any security on the updating of data member values after object creation. Anyone can come and update the object of our class without any issues:

```js
let iphone = new Product("Iphone 11", 900, "Electronics", "Apple Iphone 11", 4.5);

iphone.name = "Iphone 13";   // ✅ works — no restriction
iphone.price = -1000;        // ✅ works — even a negative price is accepted!
iphone.category = "xyz";     // ✅ works — any random value
```

This violates a fundamental OOP principle called **Encapsulation**. It says that classes are a bundle of implementation in themselves — in no case should we be required to modify data members of a class from outside the class. Otherwise, our data members can be allocated any random value without any validation.

---

## Problem with `get`/`set` Before Private Fields

The first instinct to fix this is to use `get` and `set` keywords directly on public fields and add validation there:

```js
class Product {
    constructor(builder) {
        this.name = builder.name;

        if (builder.price > 0 && typeof(builder.price) === "number") {
            this.price = builder.price; // ⚠️ calls the setter below
        } else {
            return {}; // return empty object to signal failure
        }

        this.category = builder.category;
        this.description = builder.description;
        this.rating = builder.rating;
    }

    get price() {
        return this.price; // ❌ PROBLEM: calling this.price calls the getter again!
    }

    set price(p) {
        if (p > 0) {
            this.price = p; // ❌ PROBLEM: calling this.price = p calls the setter again!
        } else {
            console.log("Invalid price");
        }
    }
}
```

### ❌ What goes wrong here — Infinite Recursion

Inside `set price(p)`, writing `this.price = p` doesn't store `p` anywhere — it calls the **setter itself again**, which calls the setter again, which calls the setter again... forever. This causes a **Maximum call stack size exceeded** error (stack overflow).

Similarly, inside `get price()`, writing `return this.price` calls the **getter itself again** infinitely.

### Why this happens

When you define `get price()` and `set price()` on a class, JS replaces the property `price` with these getter/setter functions. So whenever you write `this.price` anywhere, JS runs the getter. Whenever you write `this.price = something`, JS runs the setter. There is no actual storage location called `price` anymore — the getter and setter ARE the property.

### The Fix: Private Fields (`#`)

We need a **separate private storage location** that the getter/setter can read from and write to, without triggering themselves. That's exactly what `#price` provides.

```js
class Product {
    #price; // private storage — not accessible outside, doesn't conflict with the getter/setter name

    get price() {
        return this.#price; // ✅ reads from private storage — no recursion
    }

    set price(p) {
        if (p > 0) {
            this.#price = p; // ✅ writes to private storage — no recursion
        } else {
            console.log("Invalid price");
        }
    }
}
```

Now `this.#price` and `this.price` are two different things — `#price` is the actual private storage, and `price` is the getter/setter interface. No infinite loop.

---

## Private Fields with `#`

To resolve the encapsulation problem, we can **hide data members** from being accessed outside of the class. In JS, we can make data members **private** — accessible for read and write **only inside the class**.

To make a data member private, prepend a `#` before its declaration:

```js
class Product {
    #name;
    #price;
    #description;

    constructor(productName, productPrice, productDescription) {
        this.#name = productName;
        if (productPrice > 0 && typeof productPrice === "number") {
            this.#price = productPrice;
        }
        this.#description = productDescription;
    }

    displayProduct() {
        // ✅ private fields accessible INSIDE the class
        console.log("Product displayed", this.#name, this.#price, this.#description);
    }
}

// iphone.#price = -1000; ❌ SyntaxError — cannot access outside class
```

---

## Getters & Setters (After Private Fields)

Now that data members are private, users need a **controlled way** to get and set values. We write getter and setter methods with validation logic so that no one can allocate random values.

### Method-based Getters/Setters

```js
class Product {
    #price;

    getPrice() {
        return this.#price;
    }

    setPrice(p) {
        if (p > 0) {
            this.#price = p;
        } else {
            console.log("Invalid price");
        }
    }
}
```

### Keyword-based `get` / `set`

JS also provides dedicated `get` and `set` keywords as an alternative syntax. They are defined like functions but accessed like properties (no parentheses when calling).

```js
class Product {
    #description;

    get description() {
        console.log("Getter called");
        return this.#description;
    }

    set description(d) {
        if (d.length === 0) {
            console.log("Invalid description");
            return;
        }
        this.#description = d;
    }
}

const iphone = new Product();
iphone.description = "Apple Iphone 11"; // calls the setter — looks like property assignment
console.log(iphone.description);        // calls the getter — looks like property access
```

> **Key point:** `iphone.description = "something"` triggers the **setter**. `iphone.description` (just reading it) triggers the **getter**. No parentheses needed — they look like normal property access but run the function.

---

## Intermediate Version — Mix of Private & Public Fields

In the course of building the class, there was an intermediate stage where only `name` and `price` were private (sensitive fields), while `category`, `description`, and `rating` were still public. This shows that you don't have to make everything private — only what needs protection:

```js
class Product {
    #name;
    #price;
    category;       // public
    description;    // public
    rating;         // public

    constructor(productName, productPrice, productCategory, productDescription, productRating) {
        this.#name = productName;
        if (productPrice > 0 && typeof productPrice === "number") {
            this.#price = productPrice;
        }
        this.category = productCategory;
        this.description = productDescription;
        this.rating = productRating;
    }

    getPrice() {
        return this.#price;
    }

    setPrice(p) {
        if (p > 0) {
            this.#price = p;
        } else {
            console.log("Invalid price");
        }
    }

    addToCart() {
        console.log("Product added to cart");
    }

    removeFromCart() {
        console.log("Product removed from cart");
    }

    displayProduct() {
        console.log("Product displayed", this.#name, this.#price, this.category, this.description, this.rating);
    }

    buyProduct() {
        console.log("Product bought");
    }
}

let iphone = new Product("Iphone 11", 900, "Electronics", "Apple Iphone 11", 4.5);
console.log(iphone);

iphone.setPrice(-1000);         // "Invalid price" — setter protects #price
iphone.displayProduct();
console.log(iphone.getPrice()); // 900
```

---

## Complete Product Class (Full Evolved Version)

This is the fully evolved class — all fields private, undefined validation in constructor, method-based getters/setters for some fields, keyword-based `get`/`set` for description:

```js
class Product {
    #name;
    #price;
    #category;
    #description;
    #rating;

    constructor(productName, productPrice, productCategory, productDescription, productRating) {
        // Validate ALL params before any assignment — if any is missing, stop here
        if (
            productName === undefined ||
            productPrice === undefined ||
            productCategory === undefined ||
            productDescription === undefined ||
            productRating === undefined
        ) {
            console.log("Invalid parameters");
            return;
        }

        this.#name = productName;

        if (productPrice > 0 && typeof productPrice === "number") {
            this.#price = productPrice;
        }

        this.#category = productCategory;
        // Note: this.description calls the keyword setter below (not this.#description directly)
        this.description = productDescription;
        this.#rating = productRating;
    }

    // Method-based getter/setter for price
    getPrice() {
        return this.#price;
    }

    setPrice(p) {
        if (p > 0) {
            this.#price = p;
        } else {
            console.log("Invalid price");
        }
    }

    // Method-based getter/setter for name
    getName() {
        return this.#name;
    }

    setName(n) {
        this.#name = n;
    }

    // Method-based getter/setter for category
    getCategory() {
        return this.#category;
    }

    setCategory(c) {
        if (c === undefined) {
            console.log("Invalid category");
            return;
        }
        this.#category = c;
    }

    // Keyword-based get/set for description
    get description() {
        console.log("Getter called");
        return this.#description;
    }

    set description(d) {
        if (d.length === 0) {
            console.log("Invalid description");
            return;
        }
        this.#description = d;
    }

    // Member functions
    addToCart() {
        console.log("Product added to cart");
    }

    removeFromCart() {
        console.log("Product removed from cart");
    }

    displayProduct() {
        // this.description here calls the keyword getter for description
        console.log(
            "Product displayed",
            this.#name,
            this.#price,
            this.#category,
            this.description,
            this.#rating
        );
    }

    buyProduct() {
        console.log("Product bought");
    }
}

// Usage
let iphone = new Product("Iphone 11", 900, "Electronics", "Apple Iphone 11", 4.5);
console.log(iphone);

iphone.setPrice(-1000);          // logs "Invalid price"
iphone.displayProduct();

console.log(iphone.getPrice()); // 900

iphone.description = "";         // calls keyword setter → logs "Invalid description"
console.log(iphone.description); // calls keyword getter → logs "Getter called", returns current value
iphone.displayProduct();
```

---

## Static Keyword

The `static` keyword associates a method or property with the **class itself**, not with any instance/object.

- You **don't need to create an object** to access a static property/method.
- When the class is loaded into memory, static properties and methods are created then and there.
- Static properties are **shared across all instances** — they belong to the class, not to any object.

```js
class Product {
    static x = 10;

    constructor(name, price) {
        this.name = name;
        this.price = price;
        console.log(x);        // ❌ ReferenceError: x is not defined
                               // x alone is not a variable in scope
        // console.log(Product.x) // ✅ correct way — must use class name
    }
}

let p1 = new Product("Iphone", 1000); // ❌ throws ReferenceError because of console.log(x)

console.log(Product.x); // ✅ 10 — accessed on the class directly
// Product.x = 20;
// console.log(Product.#x); // ❌ if it were private static, this would error too
```

> **Common mistake from pasted code:** The constructor had `console.log(x)` instead of `console.log(Product.x)`. Inside the constructor, static properties are **not** automatically in scope as bare variables — you must always access them via the class name: `Product.x`.

---

## Function Constructors (Pre-ES6)

Before the `class` keyword existed in JS, to do blueprinting we only had **function constructors**. Every class we write today is actually a wrapper over this function-based mechanism.

```js
function Product(n, p, d) {
    this.name = n;
    this.price = p;
    this.description = d;

    this.displayProduct = function() {
        console.log("Name:", this.name, "Price:", this.price, "description:", this.description);
    }
}

let a = new Product("macbook", 12500, "Apple macbook");
a.displayProduct();
```

The `Product` function acts as a **function constructor**. It uses `this` to assign properties. When called with `new`, the same 4-step process happens:

1. Create an empty object
2. `this` inside the function points to that empty object; the function runs and assigns `name`, `price`, `description`, and `displayProduct` onto it
3. Set up prototypes
4. If the function returns a new object manually → return it; else return `this`

---

## Builder Design Pattern

### Problem 1: Too Many Constructor Parameters

```js
class Product {
    #name;
    #price;
    #category;
    // ...

    constructor(name, description, price, category, discount /* ... */) {
        this.name = name;
        this.price = price;
        // ...
    }
}
// Hard to remember which argument goes in which position!
// new Product("Iphone", 900, "Electronics", "Apple Iphone", 4.5)
// Was it price second or third? Category or description first?
```

Also, JS **does not support multiple constructors**, so you can't easily define `constructor(name, price)` and `constructor(name, price, category)` as separate overloads.

### Problem 2: Using Only Getters/Setters (No Custom Constructor)

One workaround is to skip the constructor entirely and use only setters:

```js
class Product {
    #name;
    #price;

    get name() { ... }
    set name(name) { ... }
    get price() { ... }
    set price(p) { ... }
}

const p = new Product(); // create empty object first
p.name = "Iphone";       // then set values
p.price = 8700;
```

But this has a **critical problem**: we cannot validate or stop object creation before it happens. If a price must be positive to create a valid product, there's no way to prevent `new Product()` from running. By the time we call `p.price = -1`, the object already exists.

> **Constructors allow pre-creation validation. Getters/setters allow controlled access after creation. We need both.**

### V1: Pass a Single Builder Object into the Constructor

Instead of many parameters, pass **one object** whose keys are the property names. Unwanted properties can simply be omitted (they'll be `undefined`):

```js
class Product {
    #name;
    #price;
    #category;

    constructor(builder) {
        this.#name = builder.name;
        if (builder.price > 0) {
            this.#price = builder.price;
        }
        this.#description = builder.description;
    }

    get name() { ... }
    set name(name) { ... }
    get price() { ... }
    set price(p) { ... }
}

const p = new Product({ name: "Iphone 11", price: 8700, description: "Apple Iphone" });
// No need to remember order — keys make it self-documenting
```

This is the **V1 of the Builder pattern** — it helps create objects in a cleaner way. But we can improve it further.

### V1 Problem: The non-static `get Builder()` approach

The next idea is to add a `Builder` getter to the `Product` class itself, so the builder object is created via `Product`:

```js
class Product {
    // ...
    get Builder() {
        // returns a builder object
    }
}
```

**Problem:** To call `product.Builder`, we need an **existing Product object**. But we need the builder to *create* the Product object in the first place. This is circular — you can't get the builder without already having a Product, and you can't build a Product without the builder.

**Solution:** Make it `static`. A static getter belongs to the **class itself**, not to any instance.

### V2: Enhanced Builder Pattern with Static Getter + Method Chaining

```js
class Product {
    #name;
    #price;
    #description;

    constructor(builder) {
        console.log("Calling Product constructor");
        this.#name = builder.name;
        if (builder.price > 0) {
            this.#price = builder.price;
        } else {
            return {}; // price invalid → return empty object to signal failure
                       // {} is an object, so new returns this empty {} instead of this
        }
        this.#description = builder.description;
    }

    displayProduct() {
        console.log("Product displayed", this.#name, this.#price, this.#description);
    }

    static get Builder() {
        class Builder {
            constructor() {
                this.name = "";        // default value for name
                this.price = 0;        // default value for price
                this.description = ""; // default value for description
            }

            setName(incomingName) {
                this.name = incomingName;
                return this; // returns the Builder object itself → enables chaining
            }

            setPrice(incomingPrice) {
                this.price = incomingPrice;
                return this;
            }

            setDescription(incomingDescription) {
                this.description = incomingDescription;
                return this;
            }

            build() {
                return new Product(this);
                // 'this' here is the Builder object
                // Product constructor receives it as 'builder'
            }
        }

        return Builder;
        // When Product.Builder is accessed, the Builder class itself is returned
        // So 'new Product.Builder()' creates a new Builder instance
    }
}

// Usage — clean and readable method chaining
const p = new Product.Builder()
                    .setName("Iphone")
                    .setPrice(1000)
                    .setDescription("Apple Iphone")
                    .build();

p.displayProduct();
```

#### How it works step by step:
- `Product.Builder` → calls the static getter → returns the `Builder` **class**
- `new Product.Builder()` → creates a new **Builder instance** with default values
- `.setName("Iphone")` → sets `this.name = "Iphone"` on the builder, returns `this` (the builder)
- `.setPrice(1000)` → sets `this.price = 1000`, returns `this`
- `.setDescription("Apple Iphone")` → sets `this.description`, returns `this`
- `.build()` → calls `new Product(this)` where `this` is the builder → Product constructor runs → Product object returned

#### Why `return this` in every setter?

Without `return this`, each setter returns `undefined`. So:
```js
new Product.Builder().setName("Iphone") // returns undefined
                     .setPrice(1000)    // ❌ TypeError: Cannot read properties of undefined
```

With `return this`, each setter hands back the **same builder object**, allowing the next call to chain on it.

#### Without chaining — verbose alternative (no `return this`):
```js
const b = new Product.Builder();
b.setName("Iphone");
b.setPrice(1000);
b.setDescription("Apple Iphone");
const p = b.build();
```

Works, but tedious. Chaining is cleaner.

---

## Object Destructuring

Doesn't matter if we create an object directly or use classes — eventually the object created is a combination of **key-value pairs**.

**Object destructuring** is a mechanism using which we try to **unpack** the object into multiple individual variables. In an object, key-value pairs are all associated to one entity — but using destructuring we can associate them to individual separate variables.

```js
const product = { name: "Iphone 14 Pro", price: 125000, category: "Mobiles" };

// Wrap variable names in {} and equate to the object
const { name, price, category } = product;
console.log(name);     // "Iphone 14 Pro"
console.log(price);    // 125000
console.log(category); // "Mobiles"
```

> Variable names **must exactly match** the keys. You cannot directly use a different name like `productName` instead of `name`.

```js
const { productName } = product; // undefined — no key called 'productName' in product
```

### Partial Destructuring

It is **not mandatory** to unpack all key-value pairs. If we only want a few of them, the syntax still works:

```js
const { name, price } = product; // category is simply ignored
```

### Aliases

To give original keys different names in the destructured variables, use **alias** syntax: `{ originalKey: aliasName }`:

```js
const { name: productName, price: productPrice, category } = product;
// productName = "Iphone 14 Pro"   ← alias for 'name'
// productPrice = 125000            ← alias for 'price'
// category = "Mobiles"             ← no alias, uses original key name
```

`productName` and `productPrice` are aliases for `name` and `price` respectively.

### Default Values

Provide a fallback value if the key doesn't exist in the object:

```js
const { name, discount = 10 } = product;
// If product has no 'discount' key → discount = 10
// If product does have 'discount' key → discount = that value
```

### Nested Object Destructuring

```js
const product = {
    name: "Iphone 14 Pro",
    price: 125000,
    category: { name: "Mobiles", categoryId: 12 }
};

// Two-step approach
const { category } = product;
const { categoryId } = category;

// One-liner — only fetches categoryId, nothing else
const { category: { categoryId } } = product;
```

### Combining Objects with Spread

Use the spread operator `...` to unpack one object's all key-value pairs inside another object:

```js
const purchasedProduct = {
    orderId: "xsya122",
    orderDate: "11/09/2024",
    ...product   // all key-value pairs from product unpacked here
};

// purchasedProduct looks like:
// { orderId: 'xsya122', orderDate: '11/09/2024', name: 'Iphone 14 Pro', price: 125000, category: 'Mobiles' }
```

---

## Array Destructuring

All destructuring logic works for arrays too. Use **square brackets** instead of curly braces.

For arrays, **names don't matter — only sequencing/position does**. Variables are allocated values by their position in the array.

```js
const [english, hindi, maths, science, sst] = [100, 100, 98, 99, 100];
// english=100, hindi=100, maths=98, science=99, sst=100
```

### Partial Array Destructuring

```js
const [english, hindi, maths] = [100, 100, 98, 99, 100];
// Only first 3 are captured; 99 and 100 are ignored
```

---

## Spread Operator & Rest Parameter

Both use `...` but serve **opposite purposes**.

### Spread Operator — Unpacks / Expands

Expands an iterable (object/array) into individual elements.

```js
// Merge multiple objects into a new one
const result = { ...o1, ...o2, ...o3 };

// Clone an object
const clone = { ...o1 };

// Update a value and create a new object at the same time
const updated = { ...o1, keyToBeUpdated: newValue };
// All keys from o1 are copied, but keyToBeUpdated is overridden with newValue

// Unpack an array inside another array
const x = [1, 2, 3];
const y = [4, 5, 6, ...x]; // [4, 5, 6, 1, 2, 3]
```

### Rest Parameter — Packs / Collects

Rest parameter is the **opposite** of spread. While spread unpacks key-value pairs, rest **packs remaining values into one unit**.

```js
// In object destructuring — collect remaining keys into one object
const { discount, ...productWithoutDiscount } = product;
// 'discount' is extracted separately
// productWithoutDiscount has everything else from product
// So product's 'discount' key is NOT part of productWithoutDiscount

// In functions — collect all arguments into an array (variadic functions)
function sum(...theArgs) {
    let total = 0;
    for (const arg of theArgs) {
        total += arg;
    }
    return total;
}

console.log(sum(1, 2, 3));    // Expected output: 6
console.log(sum(1, 2, 3, 4)); // Expected output: 10
```

> `theArgs` is technically an **array** which is combining multiple arguments into one. This is what allows a function to accept **any number of arguments** (variadic functions).

Just like using spread to unpack an object inside another object, we can do the same for arrays:

```js
const x = [1, 2, 3];
const y = [4, 5, 6, ...x]; // [4, 5, 6, 1, 2, 3]
```

> **Spread** = unpack one → many  
> **Rest** = pack many → one

---

## Quick Reference Cheat Sheet

| Concept | Syntax | Notes |
|---|---|---|
| Define a class | `class Product { }` | Blueprint for objects |
| Create object | `let p = new Product()` | Calls constructor automatically |
| Private field | `#fieldName;` | Only accessible inside the class |
| Constructor | `constructor(...) { }` | Auto-called on `new`; use for pre-creation validation |
| `this` keyword | refers to call site | In arrow functions → resolved lexically outward |
| Local var vs `this` | `let x = 3` vs `this.x` | Completely separate — don't confuse them |
| `get`/`set` without `#` | `get price() { return this.price }` | ❌ Infinite recursion — getter calls itself |
| `get`/`set` with `#` | `get price() { return this.#price }` | ✅ Reads from private storage safely |
| Method getter | `getPrice() { return this.#price }` | Called as `obj.getPrice()` |
| Method setter | `setPrice(p) { ... }` | Called as `obj.setPrice(100)` |
| Keyword getter | `get prop() { return this.#x }` | Called as `obj.prop` (no parentheses) |
| Keyword setter | `set prop(v) { this.#x = v }` | Called as `obj.prop = value` |
| Static member | `static x = 10` | Access via `ClassName.x`, not on instance |
| Static inside constructor | `console.log(Product.x)` | ❌ `console.log(x)` throws ReferenceError |
| `return {}` from constructor | `return {}` | Returns empty object — `new` respects it since `{}` is an object |
| Function constructor | `function Product(n,p) { this.name=n }` | Pre-ES6 blueprinting; same 4-step `new` process |
| Builder V1 | `new Product({ name, price })` | Single object arg — no ordering confusion |
| Non-static Builder problem | `product.Builder` | ❌ Need an object to access it — circular |
| Builder V2 | `new Product.Builder().setName().build()` | Static getter + method chaining |
| Method chaining | `return this` in setters | Returns the builder itself so next call can chain |
| Destructure object | `const { a, b } = obj` | Names must match keys exactly |
| Alias | `const { a: myA } = obj` | `myA` holds value of key `a` |
| Default value | `const { a = 5 } = obj` | Used if key is missing from object |
| Nested destructure | `const { a: { b } } = obj` | Unpacks nested key `b` |
| Spread object | `const c = { ...a, ...b }` | Merge/clone objects |
| Spread array | `const y = [...x, 4, 5]` | Expand array inline |
| Update + clone | `const c = { ...a, key: newVal }` | Clone with one key overridden |
| Rest in object | `const { x, ...rest } = obj` | `rest` = everything except `x` |
| Rest in function | `function f(...args) { }` | `args` is array of all arguments — variadic |
| Array destructure | `const [a, b, c] = arr` | Position matters, names don't |
