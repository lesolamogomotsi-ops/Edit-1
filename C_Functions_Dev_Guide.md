# C Functions — Dev Guide (Understand → Practice → Implement)
*Based on C How to Program, 9th Ed., Chapter 5*

This guide is organized as a learning loop for each topic: **Understand** the concept, **Practice** with small drills, then **Implement** a real mini-project that forces you to use it. Work top to bottom — later sections (recursion, storage classes) assume you're comfortable writing and calling functions.

---

## 0. Setup

You need a C compiler. Fastest path:

```bash
# Linux/WSL
sudo apt install build-essential

# macOS
xcode-select --install

# Compile & run any file below
gcc -std=c11 -Wall -Wextra -o prog file.c && ./prog
```

`-Wall -Wextra` matters — the compiler's warnings are how you *practice* function prototypes correctly (Section 3). Don't ignore them.

---

## 1. Why Functions Exist (Modularization)

### Understand
- Large programs are built from small, manageable pieces — **divide and conquer**.
- A function is a black box: the **caller** doesn't need to know *how* it works, only *what* it does (its interface).
- Motivations: **software reusability** (don't reinvent the wheel — use `<math.h>`, `<string.h>`, etc.) and **abstraction** (name a task once, use it everywhere).
- Rule of thumb: one function = one well-defined task. If it's doing too much, **decompose** it into smaller functions.

### Practice
1. List 3 things `printf` and `scanf` abstract away that you'd otherwise have to write by hand.
2. Take a "print a receipt" task and decompose it on paper into 4–5 named sub-tasks (`printHeader`, `printLineItem`, `printTotal`, ...) before writing any code.

### Implement
Nothing to code yet — this is the design mindset for everything below.

---

## 2. Math Library Functions (`<math.h>`)

### Understand
| Function | Description | Example |
|---|---|---|
| `sqrt(x)` | square root | `sqrt(900.0)` → `30.0` |
| `cbrt(x)` | cube root | `cbrt(27.0)` → `3.0` |
| `exp(x)` | eˣ | `exp(1.0)` → `2.718282` |
| `log(x)` | natural log | `log(2.718282)` → `1.0` |
| `log10(x)` | log base 10 | `log10(100.0)` → `2.0` |
| `fabs(x)` | absolute value | `fabs(-13.5)` → `13.5` |
| `ceil(x)` | round up | `ceil(9.2)` → `10.0` |
| `floor(x)` | round down | `floor(9.2)` → `9.0` |
| `pow(x, y)` | xʸ | `pow(2, 7)` → `128.0` |
| `fmod(x, y)` | floating-point remainder | `fmod(13.657, 2.333)` → `1.992` |
| `sin/cos/tan(x)` | trig (radians) | `sin(0.0)` → `0.0` |

All floating-point math functions return `double`, regardless of argument type.

### Practice
Write one-liners that print:
- The hypotenuse of a 3-4-5 triangle using `sqrt` and `pow`.
- `ceil` and `floor` of `-9.8` and `9.2` — predict output before running.

### Implement
```c
// mini quadratic solver — forces you to combine sqrt, pow, fabs
#include <stdio.h>
#include <math.h>

int main(void) {
    double a, b, c;
    printf("Enter a, b, c: ");
    scanf("%lf %lf %lf", &a, &b, &c);

    double discriminant = pow(b, 2) - 4 * a * c;
    if (discriminant < 0) {
        puts("No real roots.");
    } else {
        double root1 = (-b + sqrt(discriminant)) / (2 * a);
        double root2 = (-b - sqrt(discriminant)) / (2 * a);
        printf("Roots: %.2f and %.2f\n", root1, root2);
    }
}
```
Compile with `-lm` on Linux if it doesn't link: `gcc -o quad quad.c -lm`.

---

## 3. Writing Your Own Functions

### Understand
Anatomy of a function definition:
```c
return-value-type function-name(parameter-list) {
    statements
}
```
- `void` return type = returns nothing.
- `void` parameter list = takes no arguments.
- Every parameter needs an explicit type.
- **Function prototype** (declaration before `main`) tells the compiler the signature *before* it's used, so it can type-check calls:
  ```c
  int square(int number);   // prototype — note the semicolon
  ```
- The compiler checks: argument **count**, **types**, **order**, and the **return type** context all match the prototype.
- Variables declared inside a function are **local** — invisible outside it. Parameters are also local variables.
- Functions **cannot be nested** (no defining a function inside another).
- Three ways to return control: reaching the closing `}`, bare `return;`, or `return expression;`.

### Practice
1. Write prototypes (declarations only, no bodies) for:
   - a function `isEven` that takes an `int` and returns nothing useful but... (trick question — think about what return type makes sense: `int` as a boolean, 0/1).
   - a function `average` that takes 3 `double`s and returns a `double`.
2. Intentionally forget a semicolon after a prototype and compile — read the compiler error.
3. Intentionally call a function with the wrong argument type (e.g., pass a `double` where the prototype says `int`) and observe the warning from `-Wall`.

### Implement — two canonical examples from the chapter

**`square`** (single parameter, returns a value):
```c
#include <stdio.h>

int square(int number); // prototype

int main(void) {
    for (int x = 1; x <= 10; ++x) {
        printf("%d ", square(x));
    }
    puts("");
}

int square(int number) {
    return number * number;
}
```

**`maximum`** (multiple parameters):
```c
#include <stdio.h>

int maximum(int x, int y, int z); // prototype

int main(void) {
    int n1, n2, n3;
    printf("Enter three integers: ");
    scanf("%d%d%d", &n1, &n2, &n3);
    printf("Maximum is: %d\n", maximum(n1, n2, n3));
}

int maximum(int x, int y, int z) {
    int max = x;
    if (y > max) max = y;
    if (z > max) max = z;
    return max;
}
```

**Your turn:** Write `int minimum(int x, int y, int z)` from scratch without looking at `maximum`. Then write `double average(double a, double b, double c)`.

---

## 4. Argument Coercion & Mixed-Type Expressions

### Understand
- **Argument coercion**: arguments are implicitly converted to match the parameter type (e.g., passing an `int` to a function expecting `double` works automatically).
- Converting a "higher" type down to a "lower" type (e.g., `double` → `int`) can **lose data** — compilers usually warn.
- **Usual arithmetic conversions** in mixed-type expressions promote everything to the "highest" type present: `long double` > `double` > `float` > integer promotion rules.
- Know your `printf`/`scanf` format specifiers per type — mismatches are a classic source of undefined behavior:

| Type | printf | scanf |
|---|---|---|
| `double` | `%f` | `%lf` |
| `float` | `%f` | `%f` |
| `long double` | `%Lf` | `%Lf` |
| `int` | `%d` | `%d` |
| `long int` | `%ld` | `%ld` |
| `unsigned int` | `%u` | `%u` |

### Practice
Predict the output, then verify:
```c
int a = 7;
double b = 2.0;
printf("%f\n", a / b);      // int coerced to double before division
printf("%d\n", (int)(a / b)); // explicit cast truncates
```

### Implement
Write a function `double celsiusToFahrenheit(int celsius)` — call it with an `int` argument and confirm (via `printf("%f\n", ...)`) that coercion happened correctly inside the arithmetic.

---

## 5. Function-Call Stack & Stack Frames

### Understand
- The **function-call stack** (a LIFO structure) tracks every active function call.
- Each call **pushes a stack frame** containing: the **return address** (where to resume the caller) and space for that function's **local variables**.
- Returning **pops** the frame, restores control to the return address, and destroys the local variables.
- **Stack overflow**: too many nested/unreturned calls exhaust stack memory — this is a real runtime crash, and yes, it's where stackoverflow.com got its name.
- This mental model directly explains *why* local variables disappear after a function returns, and *why* deep recursion can crash a program.

### Practice
Trace by hand: for `printf("%d squared: %d\n", a, square(a));`, list the exact push/pop order of stack frames (main → square → back to main → printf → back to main).

### Implement
Nothing to code — but do this: write a recursive function with no base case (see Section 8) and run it. Watch it crash with a stack overflow / segfault. Understanding *why* it crashes is the point.

---

## 6. Headers

### Understand
A header (`.h` file) bundles function prototypes + type/constant definitions for a library. Common ones:

| Header | Contains |
|---|---|
| `<stdio.h>` | I/O functions (`printf`, `scanf`, ...) |
| `<stdlib.h>` | conversions, memory allocation, `rand`, `srand` |
| `<math.h>` | math functions |
| `<string.h>` | string processing |
| `<time.h>` | date/time functions |
| `<ctype.h>` | character testing/conversion |
| `<limits.h>` / `<float.h>` | integer/float size limits |

Your own headers use `"quotes.h"`; system headers use `<angle_brackets.h>`.

### Practice
Look up (via `man` on Linux, or online) one function from `<string.h>` and one from `<ctype.h>` you haven't used before. Write a 2-line test of each.

### Implement
Create your own header `mymath.h` declaring `int cube(int x);`, put the definition in `mymath.c`, and call it from `main.c`. This teaches multi-file compilation:
```bash
gcc main.c mymath.c -o prog
```

---

## 7. Pass-by-Value vs. Pass-by-Reference

### Understand
- **C passes all arguments by value** — the function gets a *copy*. Modifying a parameter inside the function never changes the caller's original variable.
- Pass-by-reference (via pointers) comes in Chapter 7; arrays are a special case (passed effectively by reference — Chapter 6).
- Why it matters now: expect that "changing" a parameter inside a function is a no-op from the caller's perspective, until you learn pointers.

### Practice
```c
void tryToDouble(int x) {
    x = x * 2;
}

int main(void) {
    int n = 5;
    tryToDouble(n);
    printf("%d\n", n); // predict this BEFORE running
}
```

### Implement
Write `swap(int a, int b)` that attempts to swap two values, call it from `main`, and print both variables before/after to *prove* to yourself pass-by-value doesn't swap them. (Bookmark this — you'll fix it with pointers later.)

---

## 8. Random Numbers & a Simulation

### Understand
- `rand()` (from `<stdlib.h>`) returns an integer in `[0, RAND_MAX]`.
- **Scaling & shifting** to a custom range: `int n = a + rand() % b;` where `a` = starting value, `b` = width of range.
  - Die roll: `1 + rand() % 6`
- `rand()` is **repeatable** by default (same sequence every run) — useful for debugging.
- `srand(seed)` **seeds** the generator for a different sequence each run; `srand(time(NULL))` (needs `<time.h>`) seeds from the system clock for real randomness.

### Practice
1. Run a die-roll loop without `srand` twice — confirm identical output both times.
2. Add `srand(time(NULL))` and confirm the output changes each run.
3. Simulate a coin flip (`0`/`1`) and an 8-sided die.

### Implement — full "craps" casino game (the chapter's capstone)
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

enum Status {CONTINUE, WON, LOST};

int rollDice(void);

int main(void) {
    srand(time(NULL));
    int myPoint = 0;
    enum Status gameStatus = CONTINUE;
    int sum = rollDice();

    switch (sum) {
        case 7: case 11:
            gameStatus = WON; break;
        case 2: case 3: case 12:
            gameStatus = LOST; break;
        default:
            gameStatus = CONTINUE;
            myPoint = sum;
            printf("Point is %d\n", myPoint);
            break;
    }

    while (CONTINUE == gameStatus) {
        sum = rollDice();
        if (sum == myPoint) gameStatus = WON;
        else if (7 == sum) gameStatus = LOST;
    }

    puts(WON == gameStatus ? "Player wins" : "Player loses");
}

int rollDice(void) {
    int die1 = 1 + (rand() % 6);
    int die2 = 1 + (rand() % 6);
    printf("Player rolled %d + %d = %d\n", die1, die2, die1 + die2);
    return die1 + die2;
}
```
**Extend it yourself:** add a betting system — track a `startingBalance`, subtract/add a `bet` amount based on `gameStatus`, and loop the whole game until the player quits or goes broke. This forces you to combine functions, `enum`, loops, and I/O in one program.

---

## 9. Storage Classes

### Understand
Every identifier has: **storage duration** (how long it exists in memory), **scope** (where it can be referenced), and **linkage** (visible in this file only, or across files).

- **Automatic storage duration** (`auto`, the default for locals): created on block entry, destroyed on block exit. Rarely written explicitly.
- **Static storage duration** (`static`, `extern`): exists for the entire program run.
  - **Global variables** (declared outside any function) — `extern` by default, retain value across the whole program, avoid unless truly needed (they break modularity).
  - **`static` local variables**: still scoped to their function, but *retain their value between calls* — initialized only once.

### Practice
Predict the output of calling a function twice: once with a local `int x = 0; x++;` and once with `static int x = 0; x++;`. Verify by running.

### Implement
```c
#include <stdio.h>

void counterLocal(void) {
    int count = 0;      // resets every call
    printf("local count = %d\n", ++count);
}

void counterStatic(void) {
    static int count = 0; // persists across calls
    printf("static count = %d\n", ++count);
}

int main(void) {
    for (int i = 0; i < 3; ++i) {
        counterLocal();
        counterStatic();
    }
}
```
Run it and explain in your own words, in a comment, why the two outputs differ.

---

## 10. Scope Rules

### Understand
Four scopes:
- **Function scope** — only labels (`start:`), used with `goto`/`switch`.
- **File scope** — declared outside any function; visible from declaration point to end of file (globals, function prototypes/definitions).
- **Block scope** — declared inside `{ }`; ends at the closing brace. Nested blocks can **shadow** (hide) an outer identifier with the same name.
- **Function-prototype scope** — parameter names in a prototype are cosmetic/documentation only; the compiler ignores them.

Key insight: **storage duration ≠ scope**. A `static` local variable still only has block scope (can't be referenced outside its function) even though it lives for the whole program.

### Practice
Reproduce this scoping experiment yourself (from the chapter) and annotate each line with which `x` is in play:
```c
#include <stdio.h>
int x = 1; // global

int main(void) {
    int x = 5;
    printf("%d\n", x);       // which x?
    {
        int x = 7;
        printf("%d\n", x);   // which x?
    }
    printf("%d\n", x);       // which x?
}
```

### Implement
Write three functions — `useLocal`, `useStaticLocal`, `useGlobal` — each manipulating an `x` with a different storage class/scope, call each twice from `main`, and print results before/after each call. Compare your output to your predictions.

---

## 11. Recursion

### Understand
- A recursive function calls **itself**, directly or indirectly.
- Every recursive function needs:
  1. One or more **base cases** — simple case(s) it can answer directly, no further recursion.
  2. A **recursive step** — breaks the problem into a smaller version of itself + combines with the recursive call's result.
- For termination, each recursive call **must converge toward the base case**.
- Classic risk: forgetting the base case → infinite recursion → stack overflow (ties back to Section 5).

**Factorial** (`n! = n * (n-1)!`, base case `0! = 1! = 1`):
```c
unsigned long long int factorial(int number) {
    if (number <= 1) {
        return 1;             // base case
    } else {
        return number * factorial(number - 1); // recursive step
    }
}
```

**Fibonacci** (`fib(n) = fib(n-1) + fib(n-2)`, base cases `fib(0)=0, fib(1)=1`):
```c
unsigned long long int fibonacci(int n) {
    if (0 == n || 1 == n) {
        return n;
    } else {
        return fibonacci(n - 1) + fibonacci(n - 2);
    }
}
```
⚠️ Naive recursive Fibonacci is **exponential complexity** (~2ⁿ calls) — `fibonacci(30)` is already ~a billion calls. This is a real performance trap, not just a textbook curiosity.

### Practice
1. Trace `factorial(4)` by hand: write out every call and every return value, in order.
2. Trace `fibonacci(3)` as a call tree (it should branch into `fibonacci(2)` and `fibonacci(1)`).
3. Time `fibonacci(35)` vs `fibonacci(40)` — feel the exponential blowup firsthand.

### Implement
1. Type in `factorial` and `fibonacci` exactly as above, verify against the chapter's expected output (`5! = 120`, `Fibonacci(10) = 55`).
2. Write these recursively yourself (don't peek):
   - `int sumDigits(int n)` — sum of an integer's digits.
   - `int gcd(int a, int b)` — Euclid's algorithm.
   - `unsigned long long power(int base, int exp)` — integer exponentiation.
3. **Recursion vs. iteration challenge:** rewrite `factorial` and `fibonacci` iteratively (with `for` loops). Compare speed on `fibonacci(35)` iteratively vs. recursively.

---

## 12. Recursion vs. Iteration — Decision Guide

### Understand
| | Iteration | Recursion |
|---|---|---|
| Mechanism | loop statement | repeated function calls |
| Termination | loop-continuation condition fails | base case reached |
| Overhead | low (no extra function calls) | higher (stack frame per call — time & memory) |
| When to prefer | performance-critical code, simple counted loops | problem is naturally self-similar (trees, divide-and-conquer, backtracking) and clarity matters more than raw speed |

**Any recursive solution can be rewritten iteratively.** Choose recursion when it makes the code dramatically easier to understand (tree traversal, Towers of Hanoi, backtracking search) — not by default.

### Practice
For each, decide recursion or iteration and justify in one sentence: summing an array, traversing a binary tree, computing nth Fibonacci for large n, reversing a string.

### Implement
Take your recursive `gcd` from Section 11 and write an iterative version. Confirm both give identical output for 10 random input pairs.

---

## Self-Check Checklist

Before moving to Chapter 6 (Arrays), you should be able to, without looking anything up:

- [ ] Write a function prototype and matching definition from scratch, with correct return type and parameter types.
- [ ] Explain why a variable declared inside a function disappears when the function returns (stack frames).
- [ ] Predict the output of a program mixing local, static local, and global variables of the same name.
- [ ] Generate a random integer in an arbitrary range `[a, a+b-1]` using `rand()`.
- [ ] Write a recursive function with a correct base case and recursive step, and explain why it terminates.
- [ ] Explain, in one sentence, why C is pass-by-value only, and what that means for a function trying to modify a caller's variable.

---

## Suggested Build Order (if you want one project tying it all together)

Build a **command-line dice/card game** (extend the craps game in Section 8):
1. `int rollDice(void)` — from the chapter.
2. `double calculateOdds(int point)` — math library practice.
3. `void logRoll(int roll, int total)` using a `static` counter for "rolls this session" (storage classes).
4. `unsigned long long int factorial(int n)` used to compute combinatorial odds for a "bonus round" (recursion).
5. Split it into `dice.c` / `dice.h` / `main.c` (headers, multi-file compilation).

That single project exercises every section above.
