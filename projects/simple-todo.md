---
title: Simple To-Do
layout: project
permalink: /projects/simple-todo
tags: javascript
---

I got inspired to make this while working through Wes Bos' [Javascrip30 challenge](https://javascript30.com). It's a straightforward to-do list app, but what I think is coolest is its ability to remember your items even after you've closed the window or browser by using local storage. It's got a pretty sweet sliding sidebar too that was a lot of fun to make.

Check it out [here](https://jongrim.github.io/simple-todo).

### Project code
[GitHub](https://github.com/jongrim/simple-todo)

### Languages
- Pure JavaScript (no libraries)
- CSS Flexbox

### Notable code snippet
In order to delete items that are marked as done, it's necessary to loop over the array that stores the items. This raises the issue of how to alter the array while also looping over it? While there are multiple solutions to this problem, I chose to solve it by creating a new array and pushing any item not marked as done onto the new array. Afterwards, the list is published (the `publishList` function handles writing to the DOM), and then the local storage is updated with the new list.
```javascript
function deleteSelected() {
    let newItems = [];
    for (var item of items) {
        if (item.done == false) {
            newItems.push(item);
        }
    }
    items = Array.from(newItems);
    setLocalStorage(newItems);
    publishList(newItems, itemsHTML);
}
```

### Final notes
This is one of those fun projects where at first look, it can seem challenging (for a novice such as myself), but truly there's a pretty simple solution when you break it down to granular tasks.
