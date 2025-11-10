# JavaScript & ES6 Tricky Interview Questions - Part 1

## 🎯 **What Will Be The Output? - Complete Guide**

### Table of Contents

1. [Hoisting Questions (1-10)](#hoisting-questions)
2. [Closure Questions (11-20)](#closure-questions)
3. [Scope & Context Questions (21-30)](#scope--context-questions)
4. [Async/Promise Questions (31-40)](#asyncpromise-questions)
5. [Arrow Functions Questions (41-50)](#arrow-functions-questions)

---

## Hoisting Questions

### **Question 1: Variable Hoisting with var**

```javascript
console.log(a); // What will be the output?
var a = 5;
console.log(a); // What will be the output?
```

**Output:**

```
undefined
5
```

**Explanation:**

- Due to hoisting, `var a` declaration is moved to the top of the scope
- The code is interpreted as:

```javascript
var a; // undefined by default
console.log(a); // undefined
a = 5;
console.log(a); // 5
```

- Variable declarations are hoisted but not their assignments

---

### **Question 2: Function Hoisting**

```javascript
console.log(foo()); // What will be the output?

function foo() {
  return "Hello World";
}

console.log(bar()); // What will be the output?

var bar = function () {
  return "Hello ES6";
};
```

**Output:**

```
Hello World
TypeError: bar is not a function
```

**Explanation:**

- Function declarations are completely hoisted (both declaration and definition)
- Function expressions are treated like variable assignments
- The code is interpreted as:

```javascript
function foo() {
  return "Hello World";
} // Fully hoisted
var bar; // Only declaration hoisted, undefined

console.log(foo()); // "Hello World" - function is available
console.log(bar()); // TypeError - bar is undefined, not a function

bar = function () {
  return "Hello ES6";
};
```

---

### **Question 3: Let vs Var Hoisting**

```javascript
console.log(x); // What will be the output?
console.log(y); // What will be the output?

var x = 10;
let y = 20;
```

**Output:**

```
undefined
ReferenceError: Cannot access 'y' before initialization
```

**Explanation:**

- `var` declarations are hoisted and initialized with `undefined`
- `let` declarations are hoisted but not initialized (Temporal Dead Zone)
- Variables declared with `let` cannot be accessed before their declaration line

---

### **Question 4: Complex Hoisting Scenario**

```javascript
var a = 1;
function test() {
  console.log(a); // What will be the output?
  var a = 2;
  console.log(a); // What will be the output?
}
test();
console.log(a); // What will be the output?
```

**Output:**

```
undefined
2
1
```

**Explanation:**

- Inside `test()`, `var a` is hoisted to the function scope
- This creates a local variable that shadows the global `a`
- The function is interpreted as:

```javascript
function test() {
  var a; // Hoisted declaration, undefined
  console.log(a); // undefined (local variable)
  a = 2; // Assignment
  console.log(a); // 2
}
```

- Global `a` remains unchanged

---

### **Question 5: Function Declaration vs Expression Hoisting**

```javascript
console.log(typeof foo); // What will be the output?
console.log(typeof bar); // What will be the output?

if (true) {
  function foo() {
    return 1;
  }
  var bar = function () {
    return 2;
  };
}
```

**Output:**

```
function
undefined
```

**Explanation:**

- Function declarations inside blocks are hoisted to the top of their containing scope
- Variable declarations (`var bar`) are hoisted but initialized as `undefined`
- The function expression assignment happens only when the code executes

---

### **Question 6: Hoisting with Parameters**

```javascript
var x = 1;
function test(x) {
  console.log(x); // What will be the output?
  var x = 2;
  console.log(x); // What will be the output?
}
test(3);
```

**Output:**

```
3
2
```

**Explanation:**

- Function parameters act like local variable declarations
- The parameter `x` shadows the global `x`
- Even though `var x` is declared again, it doesn't override the parameter
- The function is interpreted as:

```javascript
function test(x) {
  // x = 3 (parameter)
  // var x; // This declaration is ignored due to parameter
  console.log(x); // 3
  x = 2; // Assignment to parameter variable
  console.log(x); // 2
}
```

---

### **Question 7: Let and Const Hoisting**

```javascript
console.log(typeof a); // What will be the output?
console.log(typeof b); // What will be the output?
console.log(typeof c); // What will be the output?

let a = 1;
const b = 2;
var c = 3;
```

**Output:**

```
ReferenceError: Cannot access 'a' before initialization
```

**Explanation:**

- The error occurs at the first `console.log(typeof a)`
- `let` and `const` are in the Temporal Dead Zone before their declaration
- Even `typeof` operator cannot access them before initialization
- `var` would return `undefined` if accessed before declaration

---

### **Question 8: Hoisting in Loops**

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // What will be the output?
}

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 200); // What will be the output?
}
```

**Output:**

```
3
3
3
0
1
2
```

**Explanation:**

- `var i` has function scope, so all setTimeout callbacks share the same `i`
- By the time callbacks execute, the loop has finished and `i = 3`
- `let j` has block scope, creating a new binding for each iteration
- Each callback captures its own copy of `j`

---

### **Question 9: Hoisting with Destructuring**

```javascript
console.log(x); // What will be the output?
console.log(y); // What will be the output?

var { x, y } = { x: 1, y: 2 };
```

**Output:**

```
undefined
undefined
```

**Explanation:**

- Destructuring with `var` follows the same hoisting rules
- Variable declarations are hoisted but not the assignments
- The code is interpreted as:

```javascript
var x, y; // Hoisted declarations
console.log(x); // undefined
console.log(y); // undefined
({ x, y } = { x: 1, y: 2 }); // Assignment happens here
```

---

### **Question 10: Nested Function Hoisting**

```javascript
function outer() {
  console.log(inner()); // What will be the output?

  function inner() {
    return "inner function";
  }

  var inner = function () {
    return "inner variable";
  };

  console.log(inner()); // What will be the output?
}
outer();
```

**Output:**

```
inner function
inner variable
```

**Explanation:**

- Function declarations take precedence over variable declarations in hoisting
- The function is interpreted as:

```javascript
function outer() {
  function inner() {
    return "inner function";
  } // Function declaration hoisted
  var inner; // Variable declaration hoisted but ignored (function already exists)

  console.log(inner()); // "inner function"

  inner = function () {
    return "inner variable";
  }; // Override with function expression

  console.log(inner()); // "inner variable"
}
```

---

## Closure Questions

### **Question 11: Basic Closure**

```javascript
function outerFunction(x) {
  return function innerFunction(y) {
    return x + y;
  };
}

var addFive = outerFunction(5);
console.log(addFive(3)); // What will be the output?
console.log(addFive(7)); // What will be the output?
```

**Output:**

```
8
12
```

**Explanation:**

- `innerFunction` forms a closure over variable `x` from `outerFunction`
- When `outerFunction(5)` is called, it returns `innerFunction` with `x = 5` in closure
- Each call to `addFive` accesses the preserved value of `x = 5`
- `addFive(3)` returns `5 + 3 = 8`
- `addFive(7)` returns `5 + 7 = 12`

---

### **Question 12: Closure in Loops (Classic)**

```javascript
for (var i = 1; i <= 3; i++) {
  setTimeout(function () {
    console.log(i); // What will be the output?
  }, 100 * i);
}
```

**Output:**

```
4
4
4
```

**Explanation:**

- All three setTimeout callbacks share the same reference to variable `i`
- By the time any callback executes, the loop has completed and `i = 4`
- Each callback accesses the final value of `i` due to closure over the same variable

**Solution using closure:**

```javascript
for (var i = 1; i <= 3; i++) {
  (function (j) {
    setTimeout(function () {
      console.log(j); // Outputs: 1, 2, 3
    }, 100 * j);
  })(i);
}
```

---

### **Question 13: Closure with Object Methods**

```javascript
var obj = {
  name: "Object",
  getName: function () {
    return function () {
      return this.name;
    };
  },
};

var getNameFunction = obj.getName();
console.log(getNameFunction()); // What will be the output?
```

**Output:**

```
undefined
```

**Explanation:**

- The inner function returned by `getName` loses its context when assigned to `getNameFunction`
- When called as `getNameFunction()`, `this` refers to the global object (or `undefined` in strict mode)
- The global object doesn't have a `name` property, so `this.name` is `undefined`

**Fix using arrow function:**

```javascript
var obj = {
  name: "Object",
  getName: function () {
    return () => {
      return this.name; // Arrow function preserves 'this' from enclosing scope
    };
  },
};
```

---

### **Question 14: Multiple Closures**

```javascript
function createFunctions() {
  var result = [];
  for (var i = 0; i < 3; i++) {
    result[i] = function () {
      return i;
    };
  }
  return result;
}

var functions = createFunctions();
console.log(functions[0]()); // What will be the output?
console.log(functions[1]()); // What will be the output?
console.log(functions[2]()); // What will be the output?
```

**Output:**

```
3
3
3
```

**Explanation:**

- All three functions in the array share the same closure over variable `i`
- After the loop completes, `i = 3`
- Each function returns the final value of `i`
- This is the classic closure-in-loop problem

---

### **Question 15: Closure with Counter**

```javascript
function createCounter() {
  var count = 0;
  return {
    increment: function () {
      return ++count;
    },
    decrement: function () {
      return --count;
    },
    getCount: function () {
      return count;
    },
  };
}

var counter1 = createCounter();
var counter2 = createCounter();

console.log(counter1.increment()); // What will be the output?
console.log(counter1.increment()); // What will be the output?
console.log(counter2.increment()); // What will be the output?
console.log(counter1.getCount()); // What will be the output?
console.log(counter2.getCount()); // What will be the output?
```

**Output:**

```
1
2
1
2
1
```

**Explanation:**

- Each call to `createCounter()` creates a new closure with its own `count` variable
- `counter1` and `counter2` are independent instances with separate closures
- `counter1` increments its own `count`: 0→1→2
- `counter2` increments its own `count`: 0→1
- Each counter maintains its own private state

---

---

## Scope & Context Questions

### **Question 16: Variable Scoping with Functions**

```javascript
var x = 10;
function test() {
  console.log(x); // What will be the output?
  if (true) {
    var x = 20;
    console.log(x); // What will be the output?
  }
  console.log(x); // What will be the output?
}
test();
console.log(x); // What will be the output?
```

**Output:**

```
undefined
20
20
10
```

**Explanation:**

- Inside `test()`, `var x` is function-scoped and hoisted
- The local `var x` shadows the global `x` throughout the entire function
- The function is interpreted as:

```javascript
function test() {
  var x; // Hoisted, undefined
  console.log(x); // undefined
  if (true) {
    x = 20; // Assignment to the hoisted variable
    console.log(x); // 20
  }
  console.log(x); // 20 (same variable)
}
```

---

### **Question 17: Block Scope with Let**

```javascript
var x = 10;
function test() {
  console.log(x); // What will be the output?
  if (true) {
    let x = 20;
    console.log(x); // What will be the output?
  }
  console.log(x); // What will be the output?
}
test();
```

**Output:**

```
10
20
10
```

**Explanation:**

- `let x = 20` creates a block-scoped variable inside the `if` block
- It doesn't shadow the global `x` outside its block scope
- The global `x` is accessible before and after the block

---

### **Question 18: This Context in Different Scenarios**

```javascript
var obj = {
  name: "Object",
  regularFunction: function () {
    console.log(this.name); // What will be the output?
  },
  arrowFunction: () => {
    console.log(this.name); // What will be the output?
  },
  nestedFunction: function () {
    function inner() {
      console.log(this.name); // What will be the output?
    }
    inner();
  },
};

obj.regularFunction();
obj.arrowFunction();
obj.nestedFunction();
```

**Output:**

```
Object
undefined
undefined
```

**Explanation:**

- `regularFunction`: `this` refers to `obj` when called as method
- `arrowFunction`: Arrow functions inherit `this` from enclosing scope (global)
- `nestedFunction`: `inner()` is called without context, `this` refers to global object

---

### **Question 19: Call, Apply, and Bind**

```javascript
var person = {
  name: "John",
  greet: function (greeting, punctuation) {
    return greeting + " " + this.name + punctuation;
  },
};

var anotherPerson = { name: "Jane" };

console.log(person.greet("Hello", "!")); // What will be the output?
console.log(person.greet.call(anotherPerson, "Hi", "?")); // What will be the output?
console.log(person.greet.apply(anotherPerson, ["Hey", "."])); // What will be the output?

var boundGreet = person.greet.bind(anotherPerson, "Greetings");
console.log(boundGreet("!")); // What will be the output?
```

**Output:**

```
Hello John!
Hi Jane?
Hey Jane.
Greetings Jane!
```

**Explanation:**

- Normal method call: `this` is `person`
- `call()`: Sets `this` to `anotherPerson`, passes arguments individually
- `apply()`: Sets `this` to `anotherPerson`, passes arguments as array
- `bind()`: Creates new function with `this` bound to `anotherPerson` and first argument pre-filled

---

### **Question 20: Lexical Scoping Chain**

```javascript
var a = 1;
function outer() {
  var a = 2;
  function middle() {
    var a = 3;
    function inner() {
      console.log(a); // What will be the output?
    }
    inner();
  }
  middle();
}
outer();
```

**Output:**

```
3
```

**Explanation:**

- JavaScript uses lexical scoping (static scoping)
- `inner()` looks for `a` in its immediate enclosing scope (`middle`)
- Finds `var a = 3` in `middle()` scope
- Scope chain: inner → middle → outer → global

---

### **Question 21: Temporal Dead Zone with Let**

```javascript
console.log(typeof x); // What will be the output?
console.log(typeof y); // What will be the output?

let x = 1;
var y = 2;
```

**Output:**

```
ReferenceError: Cannot access 'x' before initialization
```

**Explanation:**

- The error occurs at the first `console.log`
- `let` variables exist in Temporal Dead Zone before initialization
- Cannot access them even with `typeof` operator
- Code execution stops at the first error

---

### **Question 22: Function Expression vs Declaration in Blocks**

```javascript
if (true) {
  console.log(typeof foo); // What will be the output?
  console.log(typeof bar); // What will be the output?

  function foo() {
    return 1;
  }
  var bar = function () {
    return 2;
  };
}
```

**Output:**

```
function
undefined
```

**Explanation:**

- Function declarations in blocks are hoisted to function scope (or global scope)
- `foo` is available throughout its containing scope
- `var bar` declaration is hoisted but assignment happens at runtime
- At the time of `console.log`, `bar` is still `undefined`

---

### **Question 23: Object Property Access and This**

```javascript
var obj = {
  a: 1,
  b: this.a + 1,
};

console.log(obj.b); // What will be the output?

function MyConstructor() {
  this.a = 1;
  this.b = this.a + 1;
}

var instance = new MyConstructor();
console.log(instance.b); // What will be the output?
```

**Output:**

```
NaN
2
```

**Explanation:**

- In object literal, `this` refers to global object, not the object being created
- `this.a` is `undefined`, so `undefined + 1 = NaN`
- In constructor function, `this` refers to the new instance being created
- `this.a = 1`, so `this.b = 1 + 1 = 2`

---

### **Question 24: Variable Shadowing with Parameters**

```javascript
var x = 1;
function test(x) {
  console.log(x); // What will be the output?
  arguments[0] = 10;
  console.log(x); // What will be the output?

  var x = 5;
  console.log(x); // What will be the output?
}
test(2);
```

**Output:**

```
2
10
5
```

**Explanation:**

- Parameter `x` receives value `2`
- `arguments[0] = 10` modifies the parameter `x` (in non-strict mode)
- `var x = 5` assigns new value to the same variable (parameter)
- Parameter and local `var` refer to the same binding

---

### **Question 25: Closure and Variable References**

```javascript
var funcs = [];
for (var i = 0; i < 3; i++) {
  funcs.push(function () {
    return i;
  });
}

console.log(funcs[0]()); // What will be the output?
console.log(funcs[1]()); // What will be the output?
console.log(funcs[2]()); // What will be the output?

// Reset
var funcs2 = [];
for (let j = 0; j < 3; j++) {
  funcs2.push(function () {
    return j;
  });
}

console.log(funcs2[0]()); // What will be the output?
console.log(funcs2[1]()); // What will be the output?
console.log(funcs2[2]()); // What will be the output?
```

**Output:**

```
3
3
3
0
1
2
```

**Explanation:**

- `var i`: All functions share the same `i` variable, final value is 3
- `let j`: Block scoping creates new `j` for each iteration
- Each function in `funcs2` captures its own copy of `j`

---

## Async/Promise Questions

### **Question 26: Basic Promise Execution Order**

```javascript
console.log("Start");

setTimeout(() => console.log("Timeout"), 0);

Promise.resolve().then(() => console.log("Promise"));

console.log("End");
```

**Output:**

```
Start
End
Promise
Timeout
```

**Explanation:**

- Synchronous code executes first: 'Start', 'End'
- Promise callbacks go to microtask queue (higher priority)
- setTimeout goes to macrotask queue (lower priority)
- Event loop processes microtasks before macrotasks

---

### **Question 27: Promise Chain Execution**

```javascript
Promise.resolve(1)
  .then((x) => {
    console.log(x); // What will be the output?
    return x + 1;
  })
  .then((x) => {
    console.log(x); // What will be the output?
    throw new Error("Error!");
  })
  .then((x) => {
    console.log("This will not run");
  })
  .catch((err) => {
    console.log("Caught:", err.message); // What will be the output?
    return 5;
  })
  .then((x) => {
    console.log(x); // What will be the output?
  });
```

**Output:**

```
1
2
Caught: Error!
5
```

**Explanation:**

- First `.then()`: receives 1, logs 1, returns 2
- Second `.then()`: receives 2, logs 2, throws error
- Third `.then()`: skipped due to error
- `.catch()`: catches error, logs message, returns 5
- Last `.then()`: receives 5, logs 5

---

### **Question 28: Async/Await vs Promise**

```javascript
async function test1() {
  console.log("A");
  await Promise.resolve();
  console.log("B");
}

function test2() {
  console.log("C");
  Promise.resolve().then(() => console.log("D"));
  console.log("E");
}

console.log("Start");
test1();
test2();
console.log("End");
```

**Output:**

```
Start
A
C
E
End
B
D
```

**Explanation:**

- 'Start' executes first
- `test1()`: logs 'A', then `await` makes function pause
- `test2()`: logs 'C', schedules 'D' for microtask queue, logs 'E'
- 'End' executes
- Microtasks run: 'B' (from await), then 'D' (from .then)

---

### **Question 29: Promise Constructor Execution**

```javascript
console.log("Start");

new Promise((resolve) => {
  console.log("Promise constructor"); // What will be the output?
  resolve();
}).then(() => {
  console.log("Promise then"); // What will be the output?
});

console.log("End");
```

**Output:**

```
Start
Promise constructor
End
Promise then
```

**Explanation:**

- Promise constructor executes synchronously
- Execution order: 'Start' → Promise constructor → 'End'
- `.then()` callback goes to microtask queue
- Microtask executes after synchronous code: 'Promise then'

---

### **Question 30: Multiple Async Operations**

```javascript
console.log("1");

setTimeout(() => console.log("2"), 0);

Promise.resolve().then(() => console.log("3"));
Promise.resolve().then(() => console.log("4"));

setTimeout(() => console.log("5"), 0);

console.log("6");
```

**Output:**

```
1
6
3
4
2
5
```

**Explanation:**

- Synchronous: '1', '6'
- Microtasks (higher priority): '3', '4'
- Macrotasks (lower priority): '2', '5'
- Event loop processes all microtasks before any macrotask

---

### **Question 31: Async Function Return Values**

```javascript
async function test() {
  return "Hello";
}

async function test2() {
  return Promise.resolve("World");
}

console.log(test()); // What will be the output?
console.log(test2()); // What will be the output?

test().then((result) => console.log("test:", result));
test2().then((result) => console.log("test2:", result));
```

**Output:**

```
Promise { <resolved>: 'Hello' }
Promise { <resolved>: 'World' }
test: Hello
test2: World
```

**Explanation:**

- Async functions always return a Promise
- `return 'Hello'` becomes `Promise.resolve('Hello')`
- `return Promise.resolve('World')` doesn't double-wrap
- Direct console.log shows Promise objects
- `.then()` callbacks show resolved values

---

### **Question 32: Promise.all vs Sequential Await**

```javascript
async function sequential() {
  console.time("sequential");
  const a = await Promise.resolve(1);
  const b = await Promise.resolve(2);
  console.timeEnd("sequential");
  return a + b;
}

async function parallel() {
  console.time("parallel");
  const [a, b] = await Promise.all([Promise.resolve(1), Promise.resolve(2)]);
  console.timeEnd("parallel");
  return a + b;
}

sequential().then((result) => console.log("Sequential result:", result));
parallel().then((result) => console.log("Parallel result:", result));
```

**Output:**

```
sequential: 0.xxx ms
Sequential result: 3
parallel: 0.xxx ms
Parallel result: 3
```

**Explanation:**

- Sequential: Waits for each Promise one by one
- Parallel: Starts both Promises simultaneously, waits for both
- `Promise.all()` is more efficient for independent async operations
- Timing shows parallel execution is faster

---

### **Question 33: Error Handling in Async/Await**

```javascript
async function test() {
  try {
    console.log("A");
    await Promise.reject("Error 1");
    console.log("B"); // Will this execute?
  } catch (error) {
    console.log("Caught:", error);
    throw "Error 2";
  } finally {
    console.log("Finally block");
  }
  console.log("After try-catch"); // Will this execute?
}

test().catch((error) => console.log("Outer catch:", error));
```

**Output:**

```
A
Caught: Error 1
Finally block
Outer catch: Error 2
```

**Explanation:**

- 'A' executes before the await
- Promise rejects, 'B' doesn't execute
- Catch block handles 'Error 1', then throws 'Error 2'
- Finally block always executes
- 'After try-catch' doesn't execute due to thrown error
- Outer catch handles the re-thrown 'Error 2'

---

### **Question 34: Promise Race Condition**

```javascript
const promises = [
  new Promise((resolve) => setTimeout(() => resolve("First"), 100)),
  new Promise((resolve) => setTimeout(() => resolve("Second"), 50)),
  new Promise((resolve, reject) => setTimeout(() => reject("Error"), 75)),
];

Promise.race(promises)
  .then((result) => console.log("Race result:", result))
  .catch((error) => console.log("Race error:", error));

Promise.allSettled(promises).then((results) => {
  console.log("AllSettled results:");
  results.forEach((result, index) => {
    console.log(`${index}:`, result);
  });
});
```

**Output:**

```
Race result: Second
AllSettled results:
0: { status: 'fulfilled', value: 'First' }
1: { status: 'fulfilled', value: 'Second' }
2: { status: 'rejected', reason: 'Error' }
```

**Explanation:**

- `Promise.race()` resolves/rejects with the first settled promise
- 'Second' completes first (50ms), so race resolves with 'Second'
- `Promise.allSettled()` waits for all promises and returns their statuses
- Shows fulfilled/rejected status for each promise

---

### **Question 35: Microtask Queue Priority**

```javascript
console.log("Start");

setTimeout(() => console.log("Timeout 1"), 0);

Promise.resolve()
  .then(() => {
    console.log("Promise 1");
    return Promise.resolve();
  })
  .then(() => {
    console.log("Promise 2");
  });

queueMicrotask(() => console.log("Microtask"));

setTimeout(() => console.log("Timeout 2"), 0);

console.log("End");
```

**Output:**

```
Start
End
Promise 1
Microtask
Promise 2
Timeout 1
Timeout 2
```

**Explanation:**

- Synchronous: 'Start', 'End'
- Microtask queue: 'Promise 1', 'Microtask', 'Promise 2'
- Macrotask queue: 'Timeout 1', 'Timeout 2'
- Event loop processes all microtasks before any macrotask

---

---

## Arrow Functions & Variable Declaration Questions

### **Question 36: Const vs Let vs Var - Hoisting Behavior**

```javascript
console.log(a); // What will be the output?
console.log(b); // What will be the output?
console.log(c); // What will be the output?

var a = 1;
let b = 2;
const c = 3;
```

**Output:**

```
undefined
ReferenceError: Cannot access 'b' before initialization
```

**Explanation:**

- `var a` is hoisted and initialized with `undefined`
- `let b` and `const c` are hoisted but not initialized (Temporal Dead Zone)
- Error occurs at `console.log(b)` - cannot access before declaration
- Code execution stops at the first error

---

### **Question 37: Const with Objects and Arrays**

```javascript
const obj = { name: "John", age: 30 };
const arr = [1, 2, 3];

obj.name = "Jane";
obj.city = "New York";
arr.push(4);
arr[0] = 10;

console.log(obj); // What will be the output?
console.log(arr); // What will be the output?

// obj = {}; // What would happen if this line was uncommented?
// arr = []; // What would happen if this line was uncommented?
```

**Output:**

```
{ name: 'Jane', age: 30, city: 'New York' }
[10, 2, 3, 4]
```

**Explanation:**

- `const` prevents reassignment of the variable, not mutation of the object/array
- Object properties can be modified, added, or deleted
- Array elements can be modified, and new elements can be added
- Uncommenting the reassignment lines would cause `TypeError: Assignment to constant variable`

---

### **Question 38: Let in Block Scope vs Function Scope**

```javascript
function test() {
  if (true) {
    let x = 1;
    var y = 2;
  }

  console.log(y); // What will be the output?
  console.log(x); // What will be the output?
}

test();
```

**Output:**

```
2
ReferenceError: x is not defined
```

**Explanation:**

- `var y` has function scope - accessible throughout the entire function
- `let x` has block scope - only accessible within the `if` block
- Outside the block, `x` is not defined
- Error occurs when trying to access `x` outside its scope

---

### **Question 39: Const Declaration and Assignment**

```javascript
const a; // What will happen here?
a = 5;

console.log(a);
```

**Output:**

```
SyntaxError: Missing initializer in const declaration
```

**Explanation:**

- `const` variables must be initialized at the time of declaration
- Cannot declare `const` variable without assigning a value
- Syntax error occurs at compilation time, code won't execute

---

### **Question 40: Temporal Dead Zone with Let and Const**

```javascript
console.log(typeof x); // What will be the output?
console.log(typeof y); // What will be the output?
console.log(typeof z); // What will be the output?

let x = 1;
const y = 2;
var z = 3;
```

**Output:**

```
ReferenceError: Cannot access 'x' before initialization
```

**Explanation:**

- `let` and `const` variables are in Temporal Dead Zone before declaration
- Cannot access them even with `typeof` operator
- Error occurs at first `console.log(typeof x)`
- Code execution stops at the first error

---

## Object Creation & Manipulation Questions

### **Question 41: Object Property Definition**

```javascript
const obj = {};

obj.a = 1;
obj["b"] = 2;
obj[3] = "three";

console.log(obj.a); // What will be the output?
console.log(obj["b"]); // What will be the output?
console.log(obj[3]); // What will be the output?
console.log(obj["3"]); // What will be the output?

console.log(Object.keys(obj)); // What will be the output?
```

**Output:**

```
1
2
three
three
['3', 'a', 'b']
```

**Explanation:**

- Numeric keys are converted to strings
- `obj[3]` and `obj['3']` access the same property
- `Object.keys()` returns keys as strings
- Keys are ordered: numeric strings first (sorted), then others in creation order

---

### **Question 42: Object Property Computed Names**

```javascript
const key1 = "name";
const key2 = "age";

const obj = {
  [key1]: "John",
  [key2]: 30,
  [key1 + key2]: "combined",
  ["get" + key1.toUpperCase()]: function () {
    return this[key1];
  },
};

console.log(obj.name); // What will be the output?
console.log(obj.age); // What will be the output?
console.log(obj.nameage); // What will be the output?
console.log(obj.getNAME()); // What will be the output?
```

**Output:**

```
John
30
combined
John
```

**Explanation:**

- Computed property names use `[]` syntax with expressions
- `[key1]` becomes `'name'`
- `[key1 + key2]` becomes `'nameage'`
- `['get' + key1.toUpperCase()]` becomes `'getNAME'`
- Method can access object properties using `this`

---

### **Question 43: Object Destructuring with Default Values**

```javascript
const obj = { a: 1, b: null, c: undefined };

const { a, b, c, d } = obj;
const { a: x, b: y = "default", c: z = "default", d: w = "default" } = obj;

console.log(a, b, c, d); // What will be the output?
console.log(x, y, z, w); // What will be the output?
```

**Output:**

```
1 null undefined undefined
1 null default default
```

**Explanation:**

- Destructuring extracts actual values, including `null` and `undefined`
- Default values only apply when the value is `undefined` (not `null`)
- `b` is `null`, so default value is not used
- `c` is `undefined`, so default value `'default'` is used
- `d` doesn't exist in object, so it's `undefined` and default value is used

---

### **Question 44: Object Method Shorthand and This**

```javascript
const obj = {
  name: "Object",
  regularMethod: function () {
    return this.name;
  },
  shorthandMethod() {
    return this.name;
  },
  arrowMethod: () => {
    return this.name;
  },
};

console.log(obj.regularMethod()); // What will be the output?
console.log(obj.shorthandMethod()); // What will be the output?
console.log(obj.arrowMethod()); // What will be the output?

const { regularMethod, shorthandMethod, arrowMethod } = obj;
console.log(regularMethod()); // What will be the output?
console.log(shorthandMethod()); // What will be the output?
console.log(arrowMethod()); // What will be the output?
```

**Output:**

```
Object
Object
undefined
undefined
undefined
undefined
```

**Explanation:**

- Regular and shorthand methods: `this` refers to the object when called as methods
- Arrow method: `this` is lexically bound to enclosing scope (global)
- When methods are extracted and called without context, `this` becomes global object
- Global object doesn't have `name` property, so returns `undefined`

---

### **Question 45: Object.assign vs Spread Operator**

```javascript
const source = { a: 1, b: { c: 2 } };
const target1 = Object.assign({}, source);
const target2 = { ...source };

source.a = 10;
source.b.c = 20;

console.log(source); // What will be the output?
console.log(target1); // What will be the output?
console.log(target2); // What will be the output?

console.log(target1.b === source.b); // What will be the output?
console.log(target2.b === source.b); // What will be the output?
```

**Output:**

```
{ a: 10, b: { c: 20 } }
{ a: 1, b: { c: 20 } }
{ a: 1, b: { c: 20 } }
true
true
```

**Explanation:**

- Both `Object.assign()` and spread operator create shallow copies
- Top-level properties are copied by value
- Nested objects are copied by reference
- Changes to `source.a` don't affect copies (primitive value)
- Changes to `source.b.c` affect copies (shared reference to nested object)

---

## Array Creation & Manipulation Questions

### **Question 46: Array Creation Methods**

```javascript
const arr1 = new Array(3);
const arr2 = new Array(1, 2, 3);
const arr3 = Array(3);
const arr4 = Array(1, 2, 3);
const arr5 = [3];
const arr6 = Array.of(3);
const arr7 = Array.from({ length: 3 });

console.log(arr1); // What will be the output?
console.log(arr2); // What will be the output?
console.log(arr3); // What will be the output?
console.log(arr4); // What will be the output?
console.log(arr5); // What will be the output?
console.log(arr6); // What will be the output?
console.log(arr7); // What will be the output?
```

**Output:**

```
[empty × 3]
[1, 2, 3]
[empty × 3]
[1, 2, 3]
[3]
[3]
[undefined, undefined, undefined]
```

**Explanation:**

- `new Array(n)` or `Array(n)` with single number creates sparse array with length n
- Multiple arguments create array with those elements
- Array literal `[3]` creates array with element 3
- `Array.of(3)` creates array with single element 3
- `Array.from({ length: 3 })` creates array with undefined elements

---

### **Question 47: Array Destructuring and Rest**

```javascript
const arr = [1, 2, 3, 4, 5];

const [a, b] = arr;
const [x, , z] = arr;
const [first, ...rest] = arr;
const [p, q, r, s, t, u] = arr;

console.log(a, b); // What will be the output?
console.log(x, z); // What will be the output?
console.log(first, rest); // What will be the output?
console.log(p, q, r, s, t, u); // What will be the output?
```

**Output:**

```
1 2
1 3
1 [2, 3, 4, 5]
1 2 3 4 5 undefined
```

**Explanation:**

- Basic destructuring extracts first two elements
- Skipping with empty space extracts 1st and 3rd elements
- Rest operator collects remaining elements into array
- More variables than elements results in `undefined` for missing values

---

### **Question 48: Array Methods and Return Values**

```javascript
const arr = [1, 2, 3];

const result1 = arr.push(4);
const result2 = arr.pop();
const result3 = arr.slice(1, 3);
const result4 = arr.splice(1, 1, "new");

console.log("Original array:", arr); // What will be the output?
console.log("push result:", result1); // What will be the output?
console.log("pop result:", result2); // What will be the output?
console.log("slice result:", result3); // What will be the output?
console.log("splice result:", result4); // What will be the output?
```

**Output:**

```
Original array: [1, 'new', 3]
push result: 4
pop result: 4
slice result: [2, 3]
splice result: [2]
```

**Explanation:**

- `push()` modifies array, returns new length
- `pop()` modifies array, returns removed element
- `slice()` doesn't modify array, returns new array with selected elements
- `splice()` modifies array, returns array of removed elements
- Operations execute sequentially, modifying the array step by step

---

### **Question 49: Array Holes and Iteration**

```javascript
const arr = [1, , 3, , 5];

console.log(arr.length); // What will be the output?
console.log(arr[1]); // What will be the output?
console.log(1 in arr); // What will be the output?

arr.forEach((item, index) => {
  console.log(`forEach: ${index} = ${item}`);
});

for (let i = 0; i < arr.length; i++) {
  console.log(`for loop: ${i} = ${arr[i]}`);
}

const mapped = arr.map((item) => item * 2);
console.log("Mapped:", mapped); // What will be the output?
```

**Output:**

```
5
undefined
false
forEach: 0 = 1
forEach: 2 = 3
forEach: 4 = 5
for loop: 0 = 1
for loop: 1 = undefined
for loop: 2 = 3
for loop: 3 = undefined
for loop: 4 = 5
Mapped: [2, empty, 6, empty, 10]
```

**Explanation:**

- Array has holes (sparse array) at indices 1 and 3
- `length` is 5, but only 3 elements exist
- `1 in arr` returns `false` - index 1 is a hole
- `forEach` skips holes, only processes actual elements
- Regular `for` loop processes all indices, holes return `undefined`
- `map` preserves holes in the resulting array

---

### **Question 50: Array Flattening and Spread**

```javascript
const nested = [1, [2, 3], [4, [5, 6]]];
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

const flattened1 = nested.flat();
const flattened2 = nested.flat(2);
const combined1 = arr1.concat(arr2);
const combined2 = [...arr1, ...arr2];
const combined3 = [arr1, arr2];

console.log("flat():", flattened1); // What will be the output?
console.log("flat(2):", flattened2); // What will be the output?
console.log("concat:", combined1); // What will be the output?
console.log("spread:", combined2); // What will be the output?
console.log("nested arrays:", combined3); // What will be the output?
```

**Output:**

```
flat(): [1, 2, 3, 4, [5, 6]]
flat(2): [1, 2, 3, 4, 5, 6]
concat: [1, 2, 3, 4, 5, 6]
spread: [1, 2, 3, 4, 5, 6]
nested arrays: [[1, 2, 3], [4, 5, 6]]
```

**Explanation:**

- `flat()` flattens one level by default
- `flat(2)` flattens up to 2 levels deep
- `concat()` combines arrays into new array
- Spread operator `...` also combines arrays
- Without spreading, arrays remain nested

---

### **Question 51: Array Reference vs Copy**

```javascript
const original = [1, 2, 3];
const reference = original;
const shallowCopy1 = [...original];
const shallowCopy2 = Array.from(original);
const shallowCopy3 = original.slice();

original.push(4);
reference.push(5);
shallowCopy1.push(6);

console.log("original:", original); // What will be the output?
console.log("reference:", reference); // What will be the output?
console.log("shallowCopy1:", shallowCopy1); // What will be the output?
console.log("shallowCopy2:", shallowCopy2); // What will be the output?
console.log("shallowCopy3:", shallowCopy3); // What will be the output?

console.log(original === reference); // What will be the output?
console.log(original === shallowCopy1); // What will be the output?
```

**Output:**

```
original: [1, 2, 3, 4, 5]
reference: [1, 2, 3, 4, 5]
shallowCopy1: [1, 2, 3, 6]
shallowCopy2: [1, 2, 3]
shallowCopy3: [1, 2, 3]
true
false
```

**Explanation:**

- `reference` points to the same array object as `original`
- Spread, `Array.from()`, and `slice()` create new array objects
- Changes to `original` affect `reference` (same object)
- Changes to copies don't affect original (different objects)
- Equality comparison checks object identity, not content

---

### **Question 52: Array Filter and Map Chaining**

```javascript
const numbers = [1, 2, 3, 4, 5, 6];

const result1 = numbers.filter((n) => n % 2 === 0).map((n) => n * 2);

const result2 = numbers.map((n) => n * 2).filter((n) => n > 5);

console.log("Filter then map:", result1); // What will be the output?
console.log("Map then filter:", result2); // What will be the output?

const result3 = numbers
  .filter((n) => {
    console.log(`Filtering: ${n}`);
    return n > 3;
  })
  .map((n) => {
    console.log(`Mapping: ${n}`);
    return n * 2;
  });

console.log("Chained result:", result3); // What will be the output?
```

**Output:**

```
Filter then map: [4, 8, 12]
Map then filter: [4, 6, 8, 10, 12]
Filtering: 1
Filtering: 2
Filtering: 3
Filtering: 4
Filtering: 5
Filtering: 6
Mapping: 4
Mapping: 5
Mapping: 6
Chained result: [8, 10, 12]
```

**Explanation:**

- Method chaining executes left to right
- First example: filter evens [2,4,6], then double each [4,8,12]
- Second example: double all [2,4,6,8,10,12], then filter >5 [6,8,10,12]
- Third example shows execution order: filter processes all elements first, then map processes filtered results
- Different order can produce different results and performance

---

---

## Advanced ES6 Features Questions

### **Question 53: Template Literals and Tagged Templates**

```javascript
const name = "John";
const age = 30;

function highlight(strings, ...values) {
  return strings.reduce((result, string, i) => {
    const value = values[i] ? `<mark>${values[i]}</mark>` : "";
    return result + string + value;
  }, "");
}

const message1 = `Hello ${name}, you are ${age} years old`;
const message2 = highlight`Hello ${name}, you are ${age} years old`;

console.log(message1); // What will be the output?
console.log(message2); // What will be the output?

const multiline = `First line
Second line
    Indented line`;
console.log(multiline); // What will be the output?
```

**Output:**

```
Hello John, you are 30 years old
Hello <mark>John</mark>, you are <mark>30</mark> years old
First line
Second line
    Indented line
```

**Explanation:**

- Regular template literal interpolates values directly
- Tagged template passes strings array and values separately to the function
- `highlight` function wraps values in HTML `<mark>` tags
- Multiline template literals preserve line breaks and indentation

---

### **Question 54: Symbol and Object Property Keys**

```javascript
const sym1 = Symbol("key");
const sym2 = Symbol("key");
const stringKey = "key";

const obj = {
  [sym1]: "symbol1 value",
  [sym2]: "symbol2 value",
  [stringKey]: "string value",
  key: "regular property",
};

console.log(obj[sym1]); // What will be the output?
console.log(obj[sym2]); // What will be the output?
console.log(obj[stringKey]); // What will be the output?
console.log(obj.key); // What will be the output?

console.log(Object.keys(obj)); // What will be the output?
console.log(Object.getOwnPropertySymbols(obj)); // What will be the output?
console.log(sym1 === sym2); // What will be the output?
```

**Output:**

```
symbol1 value
symbol2 value
string value
string value
['key']
[Symbol(key), Symbol(key)]
false
```

**Explanation:**

- Each `Symbol()` call creates a unique symbol, even with same description
- Symbols are used as unique property keys
- `Object.keys()` only returns string keys, not symbol keys
- `Object.getOwnPropertySymbols()` returns only symbol keys
- Symbol properties don't conflict with string properties

---

### **Question 55: Generators and Iterators**

```javascript
function* numberGenerator() {
  console.log("Generator started");
  yield 1;
  console.log("After first yield");
  yield 2;
  console.log("After second yield");
  return "Done";
}

const gen = numberGenerator();
console.log("Generator created");

console.log(gen.next()); // What will be the output?
console.log(gen.next()); // What will be the output?
console.log(gen.next()); // What will be the output?
console.log(gen.next()); // What will be the output?
```

**Output:**

```
Generator created
Generator started
{ value: 1, done: false }
After first yield
{ value: 2, done: false }
After second yield
{ value: 'Done', done: true }
{ value: undefined, done: true }
```

**Explanation:**

- Generator function creates an iterator, but doesn't execute immediately
- Each `next()` call executes until the next `yield` or `return`
- Yields return `{ value, done: false }`
- Return statement gives `{ value, done: true }`
- Further `next()` calls return `{ value: undefined, done: true }`

---

### **Question 56: Set and Map Collections**

```javascript
const set = new Set([1, 2, 2, 3, 3, 3]);
const map = new Map();

map.set("a", 1);
map.set("b", 2);
map.set("a", 10); // Overwrite existing key

console.log(set); // What will be the output?
console.log(set.size); // What will be the output?
console.log(set.has(2)); // What will be the output?

console.log(map.get("a")); // What will be the output?
console.log(map.size); // What will be the output?

// Using objects as keys
const key1 = {};
const key2 = {};
map.set(key1, "object1");
map.set(key2, "object2");

console.log(map.get(key1)); // What will be the output?
console.log(map.get({})); // What will be the output?
```

**Output:**

```
Set(3) { 1, 2, 3 }
3
true
10
2
object1
undefined
```

**Explanation:**

- Set automatically removes duplicates, keeps unique values only
- Map allows any type as key, including objects
- Setting same key overwrites previous value
- Object references must match exactly; `{}` creates new object reference
- `map.get({})` returns undefined because it's a different object reference

---

### **Question 57: Destructuring with Nested Objects**

```javascript
const user = {
  id: 1,
  name: "John",
  address: {
    street: "123 Main St",
    city: "Boston",
    coordinates: {
      lat: 42.3601,
      lng: -71.0589,
    },
  },
  hobbies: ["reading", "coding"],
};

const {
  name,
  address: {
    city,
    coordinates: { lat, lng },
  },
  hobbies: [firstHobby, secondHobby],
  phone = "No phone",
} = user;

console.log(name); // What will be the output?
console.log(city); // What will be the output?
console.log(lat, lng); // What will be the output?
console.log(firstHobby, secondHobby); // What will be the output?
console.log(phone); // What will be the output?
console.log(typeof address); // What will be the output?
```

**Output:**

```
John
Boston
42.3601 -71.0589
reading coding
No phone
undefined
```

**Explanation:**

- Nested destructuring extracts values from deeply nested objects
- Array destructuring works within object destructuring
- Default values apply when property is undefined
- Intermediate objects (`address`) are not assigned to variables unless explicitly named
- `address` is undefined because it was destructured but not assigned

---

### **Question 58: Default Parameters and Arguments Object**

```javascript
function test(a = 1, b = 2, c = 3) {
  console.log("Parameters:", a, b, c);
  console.log("Arguments length:", arguments.length);
  console.log("Arguments:", [...arguments]);
}

test(); // What will be the output?
test(10); // What will be the output?
test(10, 20); // What will be the output?
test(undefined, 20, 30); // What will be the output?
test(null, 20, 30); // What will be the output?
```

**Output:**

```
Parameters: 1 2 3
Arguments length: 0
Arguments: []
Parameters: 10 2 3
Arguments length: 1
Arguments: [10]
Parameters: 10 20 3
Arguments length: 2
Arguments: [10, 20]
Parameters: 1 20 30
Arguments length: 3
Arguments: [undefined, 20, 30]
Parameters: null 20 30
Arguments length: 3
Arguments: [null, 20, 30]
```

**Explanation:**

- Default parameters are used when arguments are `undefined` or not passed
- `arguments` object contains actual arguments passed, not default values
- `undefined` triggers default parameter, `null` doesn't
- Arguments length reflects actual arguments passed, not parameter count

---

### **Question 59: Rest Parameters vs Arguments**

```javascript
function oldStyle() {
  const args = Array.from(arguments);
  return args.reduce((sum, num) => sum + num, 0);
}

function newStyle(...numbers) {
  return numbers.reduce((sum, num) => sum + num, 0);
}

const arrowFunc = (...numbers) => {
  // console.log(arguments); // What would happen if this was uncommented?
  return numbers.reduce((sum, num) => sum + num, 0);
};

console.log(oldStyle(1, 2, 3)); // What will be the output?
console.log(newStyle(1, 2, 3)); // What will be the output?
console.log(arrowFunc(1, 2, 3)); // What will be the output?

function mixed(first, second, ...rest) {
  console.log("First:", first);
  console.log("Second:", second);
  console.log("Rest:", rest);
}

mixed(1, 2, 3, 4, 5); // What will be the output?
```

**Output:**

```
6
6
6
First: 1
Second: 2
Rest: [3, 4, 5]
```

**Explanation:**

- Both styles work for collecting arguments
- Rest parameters create a real array, `arguments` is array-like object
- Arrow functions don't have `arguments` object
- Rest parameters must be last in parameter list
- Rest collects remaining arguments into an array

---

### **Question 60: Class Inheritance and Super**

```javascript
class Animal {
  constructor(name) {
    this.name = name;
    console.log(`Animal created: ${name}`);
  }

  speak() {
    return `${this.name} makes a sound`;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    console.log("Dog constructor start");
    super(name);
    this.breed = breed;
    console.log("Dog constructor end");
  }

  speak() {
    const parentResult = super.speak();
    return `${parentResult} - Woof!`;
  }
}

const dog = new Dog("Buddy", "Golden Retriever");
console.log(dog.speak()); // What will be the output?
console.log(dog instanceof Dog); // What will be the output?
console.log(dog instanceof Animal); // What will be the output?
```

**Output:**

```
Dog constructor start
Animal created: Buddy
Dog constructor end
Buddy makes a sound - Woof!
true
true
```

**Explanation:**

- `super()` must be called before accessing `this` in derived class constructor
- Constructor execution order: child starts → parent executes → child completes
- `super.speak()` calls parent class method
- `instanceof` checks entire prototype chain
- Dog is instance of both Dog and Animal classes

---

## Bonus Tricky Questions

### **Question 61: Type Coercion Edge Cases**

```javascript
console.log([] + []); // What will be the output?
console.log([] + {}); // What will be the output?
console.log({} + []); // What will be the output?
console.log(true + false); // What will be the output?
console.log("5" + 3); // What will be the output?
console.log("5" - 3); // What will be the output?
console.log(null + undefined); // What will be the output?
console.log(!!null); // What will be the output?
console.log(!!undefined); // What will be the output?
console.log(!!""); // What will be the output?
console.log(!!"false"); // What will be the output?
```

**Output:**

```

[object Object]
[object Object]
1
53
2
NaN
false
false
false
true
```

**Explanation:**

- Arrays convert to empty string when coerced: `[] → ""`
- Objects convert to "[object Object]"
- Boolean arithmetic: `true = 1, false = 0`
- String + number = string concatenation
- String - number = numeric subtraction (coerces string to number)
- `null + undefined`: null→0, undefined→NaN, result is NaN
- Double negation `!!` converts to boolean
- Empty string is falsy, non-empty string "false" is truthy

---

### **Question 62: Prototypal Inheritance Gotcha**

```javascript
function Parent() {
  this.property = "parent";
}

Parent.prototype.method = function () {
  return "parent method";
};

function Child() {
  this.property = "child";
}

Child.prototype = new Parent();
Child.prototype.constructor = Child;

const child = new Child();

console.log(child.property); // What will be the output?
console.log(child.method()); // What will be the output?

delete child.property;
console.log(child.property); // What will be the output?

Child.prototype.property = "prototype property";
delete child.property;
console.log(child.property); // What will be the output?
```

**Output:**

```
child
parent method
parent
prototype property
```

**Explanation:**

- Child constructor sets its own `property = 'child'`
- Method is inherited from Parent through prototype chain
- Deleting instance property reveals prototype property (`'parent'`)
- Setting prototype property creates new property on Child.prototype
- Property lookup: instance → Child.prototype → Parent instance (used as prototype) → Parent.prototype

---

### **Question 63: Event Loop and Microtask Priority**

```javascript
console.log("1");

setTimeout(() => console.log("2"), 0);

Promise.resolve()
  .then(() => console.log("3"))
  .then(() => console.log("4"));

queueMicrotask(() => console.log("5"));

setTimeout(() => {
  console.log("6");
  Promise.resolve().then(() => console.log("7"));
}, 0);

console.log("8");
```

**Output:**

```
1
8
3
5
4
2
6
7
```

**Explanation:**

- Synchronous code first: '1', '8'
- Microtasks (Promise.then, queueMicrotask) have higher priority: '3', '5', '4'
- Macrotasks (setTimeout) execute after all microtasks: '2'
- Second setTimeout: '6', then its microtask '7'
- Event loop: sync → all microtasks → one macrotask → all microtasks → next macrotask

---

### **Question 64: Function Hoisting with var**

```javascript
var a = 1;
function test() {
  console.log(a); // What will be the output?
  if (false) {
    var a = 2;
  }
}
test();

function outer() {
  var x = 1;
  function inner() {
    console.log(x); // What will be the output?
    var x = 2;
    console.log(x); // What will be the output?
  }
  inner();
  console.log(x); // What will be the output?
}
outer();
```

**Output:**

```
undefined
undefined
2
1
```

**Explanation:**

- `var a` inside `test()` is hoisted to function scope, shadowing global `a`
- Even though `if (false)` never executes, the declaration is hoisted
- In `inner()`, local `var x` shadows outer `x` throughout the function
- Hoisting makes it: `var x; console.log(x); x = 2;`

---

### **Question 65: Final Challenge - Complex Scope and Closure**

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log("var i:", i);
  }, 100);
}

for (let j = 0; j < 3; j++) {
  setTimeout(function () {
    console.log("let j:", j);
  }, 200);
}

function createCounter() {
  let count = 0;
  return function () {
    return ++count;
  };
}

const counter1 = createCounter();
const counter2 = createCounter();

console.log(counter1()); // What will be the output?
console.log(counter1()); // What will be the output?
console.log(counter2()); // What will be the output?

(function () {
  var a = (b = 5);
  console.log("Inside IIFE - a:", typeof a, "b:", typeof b);
})();

console.log("Outside IIFE - a:", typeof a, "b:", typeof b);
```

**Output:**

```
1
2
1
Inside IIFE - a: number b: number
Outside IIFE - a: undefined b: number
var i: 3
var i: 3
var i: 3
let j: 0
let j: 1
let j: 2
```

**Explanation:**

- `var i`: All callbacks share same variable, loop finishes with i=3
- `let j`: Block scope creates new binding each iteration
- Counters have independent closures over their own `count` variables
- `var a = b = 5` is equivalent to `var a; a = b = 5` - `b` becomes global
- Inside IIFE: both `a` and `b` are accessible
- Outside IIFE: `a` is undefined (function scoped), `b` is global number

---

## 🎯 **Summary of All Questions (1-65)**

### **📊 Question Categories:**

- **Hoisting (1-10)**: var, let, const, function declarations
- **Closures (11-15)**: Basic closures, loops, counters
- **Scope & Context (16-25)**: Variable scoping, `this` binding, call/apply/bind
- **Async/Promises (26-35)**: Event loop, Promise chains, async/await
- **Variables (36-40)**: const/let/var differences, Temporal Dead Zone
- **Objects (41-45)**: Creation, destructuring, methods, copying
- **Arrays (46-52)**: Creation, methods, holes, chaining
- **Advanced ES6 (53-60)**: Templates, Symbols, Generators, Classes
- **Bonus Challenges (61-65)**: Type coercion, prototypes, complex scenarios

### **🏆 Key Learning Outcomes:**

- ✅ Deep understanding of JavaScript execution context
- ✅ Mastery of ES6+ features and their edge cases
- ✅ Event loop and asynchronous behavior
- ✅ Prototypal inheritance and scope chain
- ✅ Common interview gotchas and pitfalls

---

## Advanced Array Methods Questions

### **Question 66: Reduce Method Edge Cases**

```javascript
const numbers = [1, 2, 3, 4, 5];
const empty = [];

console.log(numbers.reduce((acc, val) => acc + val)); // What will be the output?
console.log(numbers.reduce((acc, val) => acc + val, 0)); // What will be the output?
console.log(empty.reduce((acc, val) => acc + val, 10)); // What will be the output?
// console.log(empty.reduce((acc, val) => acc + val)); // What would happen?

const result = numbers.reduce((acc, val, index, array) => {
  console.log(`Index: ${index}, Value: ${val}, Array length: ${array.length}`);
  return acc + val;
}, 0);

console.log("Final result:", result); // What will be the output?
```

**Output:**

```
15
15
10
Index: 0, Value: 1, Array length: 5
Index: 1, Value: 2, Array length: 5
Index: 2, Value: 3, Array length: 5
Index: 3, Value: 4, Array length: 5
Index: 4, Value: 5, Array length: 5
Final result: 15
```

**Explanation:**

- Without initial value, first element becomes accumulator, iteration starts from index 1
- With initial value, iteration starts from index 0
- Empty array with initial value returns the initial value
- Empty array without initial value would throw `TypeError: Reduce of empty array with no initial value`
- Reduce passes accumulator, current value, index, and original array to callback

---

### **Question 67: Array Sort Method Gotchas**

```javascript
const numbers = [10, 5, 40, 25, 1000, 1];
const strings = ["apple", "Apple", "banana", "Banana"];
const mixed = [10, "5", 40, "25"];

console.log(numbers.sort()); // What will be the output?
console.log(numbers); // What will be the output?

const numbersCopy = [10, 5, 40, 25, 1000, 1];
console.log(numbersCopy.sort((a, b) => a - b)); // What will be the output?

console.log(strings.sort()); // What will be the output?
console.log(mixed.sort()); // What will be the output?

// Date sorting
const dates = [
  new Date("2023-01-15"),
  new Date("2023-01-01"),
  new Date("2023-12-31"),
];
console.log(dates.sort()); // What will be the output?
console.log(dates.sort((a, b) => a - b)); // What will be the output?
```

**Output:**

```
[1, 10, 1000, 25, 40, 5]
[1, 10, 1000, 25, 40, 5]
[1, 5, 10, 25, 40, 1000]
['Apple', 'Banana', 'apple', 'banana']
[10, '25', 40, '5']
[Sun Jan 01 2023..., Sun Jan 15 2023..., Sun Dec 31 2023...]
[Sun Jan 01 2023..., Sun Jan 15 2023..., Sun Dec 31 2023...]
```

**Explanation:**

- Default sort converts elements to strings and compares lexicographically
- Numbers sorted as strings: '10' < '1000' < '25' < '40' < '5'
- Sort modifies original array and returns reference to same array
- Custom comparator `(a, b) => a - b` for numeric sorting
- Uppercase letters come before lowercase in ASCII
- Dates can be sorted using numeric comparison (timestamps)

---

### **Question 68: Array Find and FindIndex**

```javascript
const users = [
  { id: 1, name: "John", active: true },
  { id: 2, name: "Jane", active: false },
  { id: 3, name: "Bob", active: true },
  { id: 2, name: "Jane Duplicate", active: true },
];

const found1 = users.find((user) => user.active);
const found2 = users.find((user) => user.id === 2);
const found3 = users.find((user) => user.id === 999);

const index1 = users.findIndex((user) => user.active);
const index2 = users.findIndex((user) => user.id === 2);
const index3 = users.findIndex((user) => user.id === 999);

console.log("Found1:", found1); // What will be the output?
console.log("Found2:", found2); // What will be the output?
console.log("Found3:", found3); // What will be the output?
console.log("Index1:", index1); // What will be the output?
console.log("Index2:", index2); // What will be the output?
console.log("Index3:", index3); // What will be the output?
```

**Output:**

```
Found1: { id: 1, name: 'John', active: true }
Found2: { id: 2, name: 'Jane', active: false }
Found3: undefined
Index1: 0
Index2: 1
Index3: -1
```

**Explanation:**

- `find()` returns the first element that matches the condition
- `find()` returns `undefined` if no match found
- `findIndex()` returns the index of first matching element
- `findIndex()` returns `-1` if no match found
- Both methods stop at first match (don't continue searching)

---

### **Question 69: Array Every and Some**

```javascript
const numbers = [2, 4, 6, 8, 10];
const mixed = [2, 4, 6, 7, 10];
const empty = [];

console.log(numbers.every((n) => n % 2 === 0)); // What will be the output?
console.log(mixed.every((n) => n % 2 === 0)); // What will be the output?
console.log(empty.every((n) => n % 2 === 0)); // What will be the output?

console.log(numbers.some((n) => n > 5)); // What will be the output?
console.log(mixed.some((n) => n % 2 === 1)); // What will be the output?
console.log(empty.some((n) => n > 5)); // What will be the output?

// Short-circuit behavior
const result = numbers.every((n) => {
  console.log(`Checking: ${n}`);
  return n % 2 === 0;
});

const result2 = mixed.every((n) => {
  console.log(`Checking mixed: ${n}`);
  return n % 2 === 0;
});
```

**Output:**

```
true
false
true
true
true
false
Checking: 2
Checking: 4
Checking: 6
Checking: 8
Checking: 10
Checking mixed: 2
Checking mixed: 4
Checking mixed: 6
Checking mixed: 7
```

**Explanation:**

- `every()` returns `true` if all elements match condition
- `every()` returns `true` for empty arrays (vacuous truth)
- `some()` returns `true` if at least one element matches
- `some()` returns `false` for empty arrays
- Both methods short-circuit: `every()` stops at first false, `some()` stops at first true

---

### **Question 70: Array Includes vs IndexOf**

```javascript
const arr = [1, 2, NaN, "hello", null, undefined];

console.log(arr.includes(1)); // What will be the output?
console.log(arr.includes(NaN)); // What will be the output?
console.log(arr.includes("hello")); // What will be the output?
console.log(arr.includes(null)); // What will be the output?
console.log(arr.includes(undefined)); // What will be the output?

console.log(arr.indexOf(1)); // What will be the output?
console.log(arr.indexOf(NaN)); // What will be the output?
console.log(arr.indexOf("hello")); // What will be the output?
console.log(arr.indexOf("Hello")); // What will be the output?

const objects = [{ id: 1 }, { id: 2 }];
const obj = { id: 1 };

console.log(objects.includes(obj)); // What will be the output?
console.log(objects.includes(objects[0])); // What will be the output?
```

**Output:**

```
true
true
true
true
true
0
-1
3
-1
false
true
```

**Explanation:**

- `includes()` uses SameValueZero comparison (can find NaN)
- `indexOf()` uses strict equality (=== comparison, cannot find NaN)
- Both are case-sensitive for strings
- `indexOf()` returns first matching index or -1 if not found
- Object comparison is by reference, not by value
- `objects.includes(obj)` is false because `obj` is a different reference

---

### **Question 71: Array FlatMap and Complex Flattening**

```javascript
const sentences = ["Hello world", "How are you", "JavaScript rocks"];
const numbers = [1, 2, 3, 4];

const words = sentences.flatMap((sentence) => sentence.split(" "));
const doubled = numbers.flatMap((n) => [n, n * 2]);
const conditional = numbers.flatMap((n) => (n % 2 === 0 ? [n] : []));

console.log("Words:", words); // What will be the output?
console.log("Doubled:", doubled); // What will be the output?
console.log("Conditional:", conditional); // What will be the output?

// Comparison with map + flat
const mapThenFlat = sentences.map((s) => s.split(" ")).flat();
console.log("Map then flat:", mapThenFlat); // What will be the output?
console.log(
  "Same as flatMap?",
  JSON.stringify(words) === JSON.stringify(mapThenFlat)
); // What will be the output?

// Nested arrays
const nested = [[1, 2], [3, [4, 5]], [6]];
console.log(
  "FlatMap level 1:",
  nested.flatMap((x) => x)
); // What will be the output?
console.log("Flat level 1:", nested.flat()); // What will be the output?
```

**Output:**

```
Words: ['Hello', 'world', 'How', 'are', 'you', 'JavaScript', 'rocks']
Doubled: [1, 2, 2, 4, 3, 6, 4, 8]
Conditional: [2, 4]
Map then flat: ['Hello', 'world', 'How', 'are', 'you', 'JavaScript', 'rocks']
Same as flatMap? true
FlatMap level 1: [1, 2, 3, [4, 5], 6]
Flat level 1: [1, 2, 3, [4, 5], 6]
```

**Explanation:**

- `flatMap()` combines `map()` and `flat()` operations
- Can return arrays of different lengths (including empty arrays for filtering)
- Only flattens one level deep
- Useful for transforming and flattening in one step
- Empty arrays in the result get removed during flattening

---

## Advanced Object Methods Questions

### **Question 72: Object.keys, Object.values, Object.entries**

```javascript
const obj = {
  a: 1,
  b: 2,
  c: 3,
};

Object.defineProperty(obj, "d", {
  value: 4,
  enumerable: false,
});

const symbol = Symbol("symbol");
obj[symbol] = "symbol value";

console.log(Object.keys(obj)); // What will be the output?
console.log(Object.values(obj)); // What will be the output?
console.log(Object.entries(obj)); // What will be the output?

console.log(Object.getOwnPropertyNames(obj)); // What will be the output?
console.log(Object.getOwnPropertySymbols(obj)); // What will be the output?

// With prototype
function Person(name) {
  this.name = name;
}
Person.prototype.sayHello = function () {
  return `Hello, ${this.name}`;
};

const person = new Person("John");
person.age = 30;

console.log(Object.keys(person)); // What will be the output?
console.log(Object.values(person)); // What will be the output?
```

**Output:**

```
['a', 'b', 'c']
[1, 2, 3]
[['a', 1], ['b', 2], ['c', 3]]
['a', 'b', 'c', 'd']
[Symbol(symbol)]
['name', 'age']
['John', 30]
```

**Explanation:**

- `Object.keys/values/entries` only return enumerable string properties
- Non-enumerable properties (like 'd') are excluded
- Symbol properties are excluded from these methods
- `getOwnPropertyNames` includes non-enumerable properties
- `getOwnPropertySymbols` returns symbol properties
- Inherited properties from prototype are excluded

---

### **Question 73: Object.assign vs Spread Operator Edge Cases**

```javascript
const target = { a: 1, b: 2 };
const source1 = { b: 3, c: 4 };
const source2 = { c: 5, d: 6 };

// Object.assign
const result1 = Object.assign(target, source1, source2);
console.log("Target after assign:", target); // What will be the output?
console.log("Result1:", result1); // What will be the output?
console.log("target === result1:", target === result1); // What will be the output?

// Spread operator
const original = { a: 1, b: 2 };
const result2 = { ...original, ...source1, ...source2 };
console.log("Original after spread:", original); // What will be the output?
console.log("Result2:", result2); // What will be the output?

// With getters/setters
const sourceWithGetter = {
  get value() {
    console.log("Getter called");
    return "dynamic value";
  },
};

const assignedResult = Object.assign({}, sourceWithGetter);
const spreadResult = { ...sourceWithGetter };

console.log("Assigned:", assignedResult); // What will be the output?
console.log("Spread:", spreadResult); // What will be the output?
```

**Output:**

```
Target after assign: { a: 1, b: 3, c: 5, d: 6 }
Result1: { a: 1, b: 3, c: 5, d: 6 }
target === result1: true
Original after spread: { a: 1, b: 2 }
Result2: { a: 1, b: 3, c: 5, d: 6 }
Getter called
Getter called
Assigned: { value: 'dynamic value' }
Spread: { value: 'dynamic value' }
```

**Explanation:**

- `Object.assign` modifies the target object and returns it
- Spread operator creates a new object, leaves original unchanged
- Later sources override earlier ones for same properties
- Both trigger getters during copying
- Both create shallow copies of enumerable properties

---

### **Question 74: Object.freeze, Object.seal, Object.preventExtensions**

```javascript
const obj1 = { a: 1, b: { c: 2 } };
const obj2 = { a: 1, b: { c: 2 } };
const obj3 = { a: 1, b: { c: 2 } };

Object.freeze(obj1);
Object.seal(obj2);
Object.preventExtensions(obj3);

// Trying to modify properties
obj1.a = 10; // Frozen
obj2.a = 10; // Sealed
obj3.a = 10; // Prevent extensions

// Trying to add properties
obj1.d = 4; // Frozen
obj2.d = 4; // Sealed
obj3.d = 4; // Prevent extensions

// Trying to delete properties
delete obj1.a; // Frozen
delete obj2.a; // Sealed
delete obj3.a; // Prevent extensions

// Modifying nested objects
obj1.b.c = 20; // Frozen (shallow)
obj2.b.c = 20; // Sealed (shallow)
obj3.b.c = 20; // Prevent extensions (shallow)

console.log("Frozen obj1:", obj1); // What will be the output?
console.log("Sealed obj2:", obj2); // What will be the output?
console.log("PreventExtensions obj3:", obj3); // What will be the output?

console.log("Is frozen:", Object.isFrozen(obj1)); // What will be the output?
console.log("Is sealed:", Object.isSealed(obj2)); // What will be the output?
console.log("Is extensible obj3:", Object.isExtensible(obj3)); // What will be the output?
```

**Output:**

```
Frozen obj1: { a: 1, b: { c: 20 } }
Sealed obj2: { a: 10, b: { c: 20 } }
PreventExtensions obj3: { a: 10, b: { c: 20 } }
Is frozen: true
Is sealed: true
Is extensible obj3: false
```

**Explanation:**

- `freeze`: Cannot modify, add, or delete properties
- `seal`: Can modify existing properties, but cannot add or delete
- `preventExtensions`: Can modify and delete existing properties, but cannot add new ones
- All are shallow operations - nested objects can still be modified
- Silent failures in non-strict mode, throws errors in strict mode

---

### **Question 75: Object Property Descriptors**

```javascript
const obj = {};

Object.defineProperty(obj, "readOnly", {
  value: "Cannot change this",
  writable: false,
  enumerable: true,
  configurable: true,
});

Object.defineProperty(obj, "hidden", {
  value: "Hidden property",
  writable: true,
  enumerable: false,
  configurable: true,
});

Object.defineProperty(obj, "permanent", {
  value: "Cannot delete or reconfigure",
  writable: true,
  enumerable: true,
  configurable: false,
});

obj.readOnly = "Try to change";
obj.hidden = "Modified hidden";
delete obj.permanent;

console.log("ReadOnly:", obj.readOnly); // What will be the output?
console.log("Hidden:", obj.hidden); // What will be the output?
console.log("Permanent:", obj.permanent); // What will be the output?

console.log("Keys:", Object.keys(obj)); // What will be the output?
console.log("Own property names:", Object.getOwnPropertyNames(obj)); // What will be the output?

const descriptor = Object.getOwnPropertyDescriptor(obj, "readOnly");
console.log("ReadOnly descriptor:", descriptor); // What will be the output?
```

**Output:**

```
ReadOnly: Cannot change this
Hidden: Modified hidden
Permanent: Cannot delete or reconfigure
Keys: ['readOnly', 'permanent']
Own property names: ['readOnly', 'hidden', 'permanent']
ReadOnly descriptor: { value: 'Cannot change this', writable: false, enumerable: true, configurable: true }
```

**Explanation:**

- `writable: false` prevents value changes
- `enumerable: false` excludes from `Object.keys()` and `for...in` loops
- `configurable: false` prevents deletion and descriptor changes
- `Object.getOwnPropertyNames()` includes non-enumerable properties
- `getOwnPropertyDescriptor()` returns the property descriptor object

---

### **Question 76: Object Prototype Chain**

```javascript
const animal = {
  type: "Animal",
  speak() {
    return `${this.name} makes a sound`;
  },
};

const dog = Object.create(animal);
dog.name = "Buddy";
dog.breed = "Golden Retriever";

console.log(dog.speak()); // What will be the output?
console.log(dog.type); // What will be the output?

console.log(dog.hasOwnProperty("name")); // What will be the output?
console.log(dog.hasOwnProperty("type")); // What will be the output?
console.log(dog.hasOwnProperty("speak")); // What will be the output?

console.log("name" in dog); // What will be the output?
console.log("type" in dog); // What will be the output?
console.log("speak" in dog); // What will be the output?

console.log(Object.getPrototypeOf(dog) === animal); // What will be the output?

// Changing prototype property
animal.type = "Modified Animal";
console.log(dog.type); // What will be the output?

// Shadowing prototype property
dog.type = "Dog";
console.log(dog.type); // What will be the output?
console.log(animal.type); // What will be the output?
```

**Output:**

```
Buddy makes a sound
Animal
true
false
false
true
true
true
true
Modified Animal
Dog
Modified Animal
```

**Explanation:**

- `Object.create()` creates object with specified prototype
- Methods and properties are inherited from prototype chain
- `hasOwnProperty()` only returns true for own properties
- `in` operator checks both own and inherited properties
- Changes to prototype affect all objects that inherit from it
- Assigning to inherited property creates own property (shadowing)

---

### **Question 77: Object Comparison and Equality**

```javascript
const obj1 = { a: 1, b: 2 };
const obj2 = { a: 1, b: 2 };
const obj3 = obj1;

console.log(obj1 == obj2); // What will be the output?
console.log(obj1 === obj2); // What will be the output?
console.log(obj1 === obj3); // What will be the output?

// JSON comparison
console.log(JSON.stringify(obj1) === JSON.stringify(obj2)); // What will be the output?

// Custom comparison
function deepEqual(obj1, obj2) {
  const keys1 = Object.keys(obj1);
  const keys2 = Object.keys(obj2);

  if (keys1.length !== keys2.length) {
    return false;
  }

  for (let key of keys1) {
    if (obj1[key] !== obj2[key]) {
      return false;
    }
  }

  return true;
}

console.log(deepEqual(obj1, obj2)); // What will be the output?

// Object with different property order
const obj4 = { b: 2, a: 1 };
console.log(JSON.stringify(obj1) === JSON.stringify(obj4)); // What will be the output?
console.log(deepEqual(obj1, obj4)); // What will be the output?
```

**Output:**

```
false
false
true
true
true
false
true
```

**Explanation:**

- Object equality compares references, not content
- `obj1` and `obj2` are different objects with same content
- `obj3` is a reference to the same object as `obj1`
- JSON comparison works for simple objects with same property order
- Custom comparison can check content equality
- Property order affects JSON.stringify but not our deepEqual function

---

**Total Questions Now: 77 Complete JavaScript/ES6 Interview Questions!**
