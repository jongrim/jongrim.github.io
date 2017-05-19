---
title: WarGames Hangman
layout: project
permalink: /projects/wargames-hangman
tags: javascript
---
Ah, who doesn't love the classic game, hangman? For my version, I chose to use the theme of WarGame, the classic 80's movie about how a teenage hacker almost starts World War III. My design goal was to make it somewhat like the play was actually playing against the supercomputer, WOPR, so you'll notice a fun intro where letters are written one at a time like in the movie. Play it [live](https://jongrim.github.io/wargames-hangman).

### Project Code
[GitHub](https://github.com/jongrim/wargames-hangman)

### Languages
- JavaScript (no libraries)
- HTML / CSS

### Notable Code
One of the biggest challenges others seemed to have with this program was figuring out how to continuously write and update the solution string that showed what letters had been correctly guessed. I believe I came up with an elegant and straight forward approach, outlined below:
```javascript
Game.prototype.rewriteSolution = function () {
    let result = '';
    let ans = this.puzzleAnswer;
    for (let i = 0; i < ans.length; i++) {
        if (/[a-z]/i.test(ans[i])) {
            if (this.lettersGuessed.indexOf(ans[i].toUpperCase()) > -1) {
                result = result.concat(`${ans[i]}&nbsp`);
            } else {
                result = result.concat(`_&nbsp`);
            }
        } else {
            if (ans[i] === ' ') {
                result = result.concat('&nbsp&nbsp');
            } else {
                result = result.concat(`${ans[i]}&nbsp`);
            }
        }
    }
    return result;
}
```

This method is defined on the Game object so it is made available to run via that object. This particular function is run at the start of the game and after each guess. For each execution, a `result` string is declared which will hold the value that is ultimately printed to the screen. I also set the puzzle answer to a local variable for ease of reference. The for block within the method works as such:

*For each character of the puzzle answer*
- Determine if the character is a letter (done using a simple RegEx check)
- If yes, check if the letter has been guessed by checking the objects `lettersGuessed` property
- If guessed, write the letter to the output string. If not, write a '_ ' (I use the HTML encoded &nbsp to avoid rendering issues)
- If the character is not a letter, check if it is a space character, and if so, substitue the proper HTML code
- Otherwise, just write the character to the string (this means it's some form of punctuation)

### Final Notes
Hangman itself is a simple game, but I think I showed how you can still put some meaningful thought into the design and structure of the overall program. I worked hard to make sure the program separated concerns properly, so I'd encourage interested parties to read the [source code](https://github.com/jongrim/wargames-hangman). I also coded my own console writer simulation which was kind of fun and used Promise objects to get the writing sequences to happen properly!
