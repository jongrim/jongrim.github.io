---
layout: post
title: Useful JavaScript Array Methods
tags: javascript code
---
I was recently pair programming with another local dev who easily has 10 years of coding experience on me (shoutout to the Atlanta Software Craftsmanship meetup). While working through the coding challenge, I was surprised that I was able to introduce him to (or maybe remind him of) the `filter` method for arrays.

Not gonna lie, it was pretty confidence inspiring.

I got a kick out of being able to contribute something while working with a more seasoned professional, but I also came away thinking a couple of things:

- You don't have to know everything to be a good developer.
    - He was clearly the better programmer. He was able to better articulate the concepts of the problem we were working on, and he better defined the problem space than I did. It's also worth noting that he had an alternative way of filtering the array, it was just a bit more verbose.

- Just because someone doesn't know something, it doesn't make them stupid.
    - We as people can be really bad about assuming everyone has learned 'the basics'. But what are the basics? Who gets to define that? If the situation were reversed, how easy would it be for a more senior person to look down on a junior because they didn't think of a particular method. The truth is, people learn on different paths, and there's *a lot* to learn, so it's very easy for someone not to know something you consider basic. When I pointed out the method we could use, the other programmer was happy to learn something new, and I was pretty excited I was able to call out something to someone who had been coding for so long. We should all try to be mindful next time we say something is 'simple' or before we respond with 'you don't know that?'.

So in that spirit, I wanted list out and go through a few methods that I have found useful so far in my learning.

## Adding and Removing Things to an Array
It's very common to need to add or remove things from an array, and you've got a few ways to accomplish this. Some good methods to know include:
- `push`
- `pop`
- `shift`
- `unshift`

```javascript
let myArray = ['this', 'that', 'the other'];
myArray.push('another'); // adds the value to the end of the array
myArray.pop(); // removes and returns the value at the end of the array
myArray.shift(); // removes and return the value at the beginning of the array
myArray.unshift('this'); // adds the value to the first position of the array
console.log('myArray'); // ['this', 'that', 'the other']
```

Push and pop both alter the value at the end of the array. More specifically, if *n* is the length of the array, they affect the value at index *n*-1. Think about it as though you're *pushing* something onto a stack of things, and then as if you're *popping* it back off. It makes sense that after you've pushed or popped something, your array has changed.

Shift and unshift are the same, but they work on the front of the array, or on index 0. Their naming isn't as appropriate in my opinion because my natural instinct would be to *shift* something into the first spot, and *unshift* it away, but strangely I wasn't consulted about the naming. Instead, try to remember that you shift something out of the first spot and then can undo it with unshift.

It's also worth remembering that shifting and unshifting is a potentially costly method. The entire array is altered since everything has to be re-indexed. This isn't an issue for smaller arrays but can introduce performance issues with very large sizes or where it's done many times.

## Making Copies or Large Scale Deletions / Insertions
- `slice`
- `splice`

I often mentally group slice and splice together, though they really are fairly different. Slice can be used to make a shallow copy of all or part of an array, whereas splice can perform large-scale deletions or insertions for an array.

```javascript
/* slice example */
let myArray = ['coffee', 'tea', 'soda'];
let copy = myArray.slice(); // copies the whole array to our variable 'copy'
copy = myArray.slice(1); // starts the copy at index 1 and goes to the end
console.log(copy); // ['tea', 'soda']
copy = myArray.slice(0, 1); // begins at 0, and ends before (not including) 1
console.log(copy); // ['coffee']
```
Slice doesn't alter the original array, but instead gives you a copy of that array. This can be useful if you need a copy to edit while looping through the array (remember, never alter the thing you're looping on or bad things can happen!). If the array contains object references, the copied value will reference the same object. This means that if the object is updated, the change will be reflected in the original and in the copied version of the array (this is what's meant by a shallow copy).

Where slice doesn't alter the original array, **splice** is all about changing it.

```javascript
/* splice example */
let myArray = ['jon', 'ari', 'nate', 'caitlin', 'briana', 'andrea'];
let deletedItems = myArray.splice(0, 2); // .splice(startPosition, deleteCount)
console.log(myArray); // ['nate', 'caitlin', 'briana', 'andrea']
console.log(deletedItems); // ['jon', 'ari']
myArray.splice(2, 0, 'jon', 'ari');
/* .splice(startPosition, deleteCount, item1, item2, ...) */
console.log(myArray); // ['nate', 'caitlin', 'jon', 'ari', 'briana', 'andrea']
```
Splice can do some crazy things where you can delete large parts, specific parts, or you can add in new parts to an array, all at the same time! I haven't personally had too much reason to use splice much, but it's handy to know in case you need to do a lot of altering to an array. It's also possible to supply a negative number for the start position of the method call, and this will cause the operation to start that many from the end of the array (though it still counts forward from there).

## Every and Some
Arrays have two methods that provide extended checking for each element within the array, `every` and `some`. Each takes a function as an argument to perform a callback. Like `&&` returns true only when both sides of the operator are true, `every` will only return true if the function returns true for every element (great name!). Similarly, `some` is like `||` in that if any element returns true, the whole thing is true. If the first element tested returned true then `some` would stop looking at elements.

```javascript
[12, 5, 4].every(function(e){return e > 10}); // false
[12, 5, 4].some(function(e){return e > 10}); // true
[12, 5, 4].every(e => e < 50); // arrow functions are great here!
```

## forEach, map, filter, and reduce
These, in my mind, are the heavy hitters. They can be very versatile and powerful, and really help with data transformation. A great technique is to chain the calls together, and I'll cover an example of that shortly.

Starting off, `forEach` and `map` are very similar. Each will apply a function of your choice to *each* element of an array. The big difference is that `map` will return the new array when complete. `forEach` on the other hand alters the array in place.

Imagine a scenario where we have an array of person objects that have this structure:
```javascript
var person = {
    firstName = 'Jon';
    lastName = 'Grim';
}
```
But what we really need later on is an array of their full name as a single string. Here's how we could build that array using `forEach` and `map`:
```javascript
/* assume we have an array called people with a bunch of person objects */
people.forEach(person => {
    person.fullName = person.firstName + ' ' + person.lastName
});
/* Note that forEach returns undefined in all cases, so we're forced to 
store the value some other way. We could have also declared another array 
before the call and then pushed the names onto that. */

let names = people.map(person => person.firstName + ' ' + person.lastName);
/* map returns an array of the new values so we can easily capture it to use 
immediately */
```

We can extend the example by introducing the `filter` method which gives us the ability to apply some test to each element of an array and return only those for which the test holds true. Let's say that now we first want to identify which people have a last name starting with 'G', and then we want to return their full name. We can start to chain our methods together for something pretty cool.

```javascript
let peopleWithLastNameG = people
    .filter(person => person.lastName[0] === 'G')
    .map(person => person.firstName + ' ' + person.lastName);
```

In this example, we first run `filter` on the array to check if our test is true for each element. This function call returns an array which we chain to the `map` call to then perform our data transformation that we want for that select group.

**But wait, there's more.**

Now we can also bring in `reduce`. `reduce` has this awesome ability to remember values as it moves through the array, so the end result is some combination or comparison that every element contributed to or was considered for. For example, let's say our person object now has an age property - `person.age` - which holds their age as an integer. Now, we can add on to our above example to find the sum of the ages of all people whose last name starts with 'G'.

```javascript
let totalAge = people
    .filter(person => person.lastName[0] === 'G')
    .map(person => person.age)
    .reduce((totalAge, curAge) => totalAge + curAge, 0);
```

In this example, we use filter in the same way to first pair down our list to just those whose last name starts with 'G'. Then, we pass the result to map to pull out each person's age property and return an array of just those numbers to `reduce`. From there, `reduce` takes two arguments. The first is an accumulator value which will store our ever growing sum, and the second is the current element of the array. We also provide `reduce` with an initial starting value for clarity, though this is an optional argument. If we had omitted this, the `reduce` function would have used the first element of the array as the initial value and would have begun running our supplied function at index 1. This would work okay in this case, but it's not always what's wanted. After all this, we now have the total age of every person with a last name starting with 'G'. There's a good bit more you can do with `reduce` and I highly recommend checking out the MDN docs for it.

---

## Additional Resources
- [MDN: Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
