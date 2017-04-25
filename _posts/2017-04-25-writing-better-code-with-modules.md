---
layout: post
title:  "Writing Better Code with Modules"
categories: javascript design patterns
---

I was recently introduced to modular JavaScript and it has changed my worldview. 

**I have seen the light, y'all.**

Before I was just a newbie (well, still am), declaring everything in the global scope because I didn't know any better. Now, my projects have a clean(er) design, and my projects benefit from modules that group code and provide easy APIs.

You too can harness this magic and write better code.

But first, *why* should you even care about this? Take for example, the code below (from a recent project I wrote before diving into modules):

```javascript
const itemsHTML = document.querySelector('.items'),
      addItemForm = document.querySelector('.add-items'),
      openSidebarButton = document.querySelector('.openSide'),
      closeSidebarButton = document.querySelector('.closeSide'),
      sidebar = document.querySelector('.sidebar'),
      checkAllButton = document.querySelector('#checkAll'),
      deleteButton = document.querySelector('#delete'),
      resetButton = document.querySelector('#reset');

var items = JSON.parse(localStorage.getItem('items')) || [];

function addItem(e) {
    //add the item to items
    e.preventDefault();
    const text = (this.querySelector("[name='item']")).value;
    let item = {
        text,
        done: false
    }
    items.push(item);
    
    //create the HTML for the item
    publishList(items, itemsHTML);

    //store list locally
    setLocalStorage(items);
    
    this.reset();
}
```

At first glance, this doesn't seem too bad. Especially if you're a newer programmer like me, you may not see anything wrong with writing code this way. After all it:
1. works, and
2. doesn't look half bad visually (consistent indenting, naming style, curly braces, etc.)

But let's go a little deeper and really nitpick. Some issues that I can identify are:
1. It's not immediately clear what it does, or how the bits are related, if they are at all
2. Everything is declared in the global scope!
3. It's not too bad with the above snippet, but writing code like above makes it pretty easy to violate some principles like DRY [(Don't Repeat Yourself)](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)


I'm sure more could be added to this list, but these are the things that stand out to me most. Because there is no strong sense of organization to the code, it makes it unclear how it should be read and interpreted. 

Enter modules, the organizers of code.

Soon, we'll explore how modules can help provide organization for related code, abstract away unnecessary details, and provide an interface for the bits we want to reuse. But first, a quick aside on function declarations and expressions.

## Declaration vs Expression

In JavaScript, there is an important difference between a function **declaration** and a function **expression**. To confuse matters, the two have almost identical syntax. The difference comes in how they are used.

```javascript
function myFunc() {
    //do some stuff
} // this is a function declaration

var myFunc = function() {
    //do some stuff
} // this is a function expression
```

The former will *declare* a function that will from then on exist within its surrounding scope (in this case global). This declaration is actually hoisted to the top along with all other declarations, so it is available to use immediately. While I don't think it's good style, you could actually write calls to this function *above* where it is written in the source code.

The second function is an expression that is assigned to the variable `myFunc`. The key difference is that this function is **not** hoisted, and is executed in order by the engine when the code is ran. This means that we cannot make use of this function until the engine has reached this point in the code, at which point it will assign the function, and its contents, to the variable for later use. From then on, we may invoke the function by adding ( ) behind the variable name, e.g. `myFunc()`.

The two are very similar, in that the end-result is a function that can be executed via a keyword followed by ( ). The biggest difference is *when* that function becomes available for use within our code.

## Immediately Invoked Function Expressions

This leads to the next step of **Immediately Invoked Function Expressions** (or **IIFE**), which work just as the name implies. First, we create a function expression, and then we immediately invoke that function to execute its code. Syntactically, this is very similar to a normal function expression, and takes the form of:
```javascript
(function myFunc() {
    //do some stuff
})();
```
Note how the function has been wrapped in an outer set of parenthesis, and then immediately followed by another set of parenthesis. By wrapping the whole function in parenthesis, the parser is tricked into expecting a function expression rather than a declaration as is normal when the keyword `function` is encountered. Then, the trailing ( ) makes the engine immediately execute the function, and its contents.

While not totally obvious, this has powerful implications. Namely, it gives the programmer the ability to create a private scope, complete with variables and functions, and then assign the results to a single global variable. This is the basis of the module pattern.

## Modules

As previously stated, modules take advantage of how JavaScript handles scope and function expressions. There is a key term to understanding JavaScript scope that I've neglected using till now to avoid confusion, which is *closure*. This is another topic that can stand alone, so I won't dive deeply, but know that in terms of scope, closure allows us to continue to access a function's scope even after it was executed.

So now, returning to our original goal of understanding how modules help write better code, let's look at an example. This comes from a program I wrote immediately after the first example, and after I had learned about modules:

```javascript
var customerSet = (function () {
    //private variables
    var customers = [];
    var Customer = function (name, credit = 0.00, orderCount = 0, redemptionEligible = false) {
        this.name = name;
        this.credit = credit.toFixed(2);
        this.orderCount = orderCount;
        this.redemptionEligible = redemptionEligible;
    };

    //DOM elements
    const customerTable = document.querySelector('#customerTable');
    const searchBar = document.querySelector('.search-bar');

    //Bind event listeners    
    searchBar.addEventListener('change', findMatches);    
    searchBar.addEventListener('keyup', findMatches);    

    function findMatches() {
        let matchedArray = customers.filter(customer => {
            let matchValue = new RegExp(this.value, 'gi');
            return customer.name.match(matchValue);
        })
        makeTable(matchedArray);
    }
    
    function newCustomer (name, credit, orderCount, redemptionEligible) {
        return new Customer(name, credit, orderCount, redemptionEligible);
    };

    function addCustomer (customer) {
        customers.push(customer);
    };

    function getCustomerByName (name) {
        return customers.find(customer => customer.name == name);
    };

    function setRedemptionStatus (customer, status) {
        customer.redemptionEligible = status;
    };

    return {
        newCustomer: newCustomer,
        addCustomer: addCustomer,
        getCustomerByName: getCustomerByName
    }
})();
```
Some things I want to point out about the code:
1. The entire chunk of code is actually just an IIFE that gets assigned to the variable `customerSet`
2. The contents of the outer function include variables, a constructor for new `Customer` objects, some code to add event listeners to certain elements on the page, and some functions that perform various related tasks
3. The returned product of the IIFE is an object with references to only a subset of the function's contents

The end result is that we have **one** variable in our global scope which now has the ability to access the inner scope of the function via the references that I chose to return. These references are what we refer to as the API. So for example, if somewhere else in other code, I needed to create a new `Customer`, I could do so by writing the following code:

```javascript
var johnDoe = customerSet.newCustomer('John', 0.00, 0, false);
```

This is a great benefit of modules, and it brings me back to the original three points I raised of how we could improve on the first example.

### Modules provide organization and group related code
The power of modules is best harnessed when you use them to bundle related code. It wouldn't make sense for me to package code in the `customerSet` example that isn't related to working with the customer data. Instead, I can create multiple modules for all the different bits of functionality needed throughout an application.

### Modules keep our namespace clean
Modules give us a way to hide unnecessary details and to isolate variables and functions from other code, thus preventing potential naming collisions. Instead, we get one variable in our global scope, and we can reference the returned references using a convenient dot notation. These publicly available references make up the modules API, and allow other modules and code to use its functionality. This frees the developer to determine what parts of a module should be publicly visible, and which parts should be private.

### Modules are a great design tool
We can use modules to build reusable bits of code. This helps us follow principles such as DRY, and makes the code more maintainable. We can build a module that provides a specific piece of functionality, and then leverage that everywhere else, rather than recreating it each time.

---

I hope this has been informative, and you can see how modules can really help you write cleaner, better code. This was a very brief intro to the topic. There are many more facets to consider, and there are even variations on modules that you can learn about (for instance, my example above is what's called the Revealing Module pattern). Additionally, modules are just one of many different design patterns out there.

## Additional Resources and References
For further reading check out these resources:
- [Ben Alman blog post on IIFE](http://benalman.com/news/2010/11/immediately-invoked-function-expression/)
- [Learning JavaScript Design Patterns by Addy Osmani](https://addyosmani.com/resources/essentialjsdesignpatterns/book/)
- [You Don't Know JS by Kyle Simpson](https://github.com/getify/You-Dont-Know-JS)
