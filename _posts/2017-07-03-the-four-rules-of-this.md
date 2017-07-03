---
layout: post
title: The Four Rules of This
tags: JavaScript
---

I'm going to print these out, laminate them, and never worry about *this* again.

- **Constructor binding** - when the call site is a constructor call using the `new` keyword, `this` will refer to the new object created (see below notes on new keyword).
- **Explicit binding** - when the call site uses `.call()` or `.bind()`, `this` will refer to the object parameter passed.
- **Implicit binding** - when the call site is an object calling a reference to a property function, `this` will refer to the calling object.
- **Default binding** - when the call site is a function reference with no binding, `this` will refer to global scope if not in strict mode. In strict mode, `this` will be undefined.
