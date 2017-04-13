---
title: Meanie-Genie
layout: project
permalink: /projects/meanie-genie
tags: javascript
---
Inspired by a puzzle presented as part of the [Effective Thinking Through Mathematics course](https://www.edx.org/course/effective-thinking-through-mathematics-utaustinx-ut-9-01x-0), the game challenges the user to break the riddle of how to correctly pick a stone through a clever process of elimination.

Play it [here](https://jongrim.github.io/meanie-genie).

### Project code
[GitHub](https://github.com/jongrim/meanie-genie)

### Languages
- Pure JavaScript (no libraries) including recent ES6 introduced syntax such as `const`, `let`, and arrow functions
- CSS Flexbox and transformations

### Notable code snippet
The `setStonesActive` function is called after the player has selected stones to be weighed. The game must eliminate the stones that weigh less and can be eliminated from the selection process. This function hides every stone element, and then determines what stones are still a part of the game by examining which have a valid `id` property.
```javascript
function setStonesActive() {
    jewels.forEach(jewel => { jewel.classList.add('hidden') });
    let stones = gameState.stones
    let activeStones = [];
    for (var i = 0; i < stones.length; i++) {
        if (stones[i].id != null) {
            document.querySelector(`#jewel${stones[i].id}`).classList.remove('hidden');
            activeStones.push(stones[i]);
        }
    }
    return activeStones;
}
```

### Final notes
Overall, I'm pretty happy with this game and think the puzzle is lots of fun! It's the first big-ish project I tackled in JavaScript, and I'm pretty proud I could complete it after having only studied JavaScript for about a week and a half!
