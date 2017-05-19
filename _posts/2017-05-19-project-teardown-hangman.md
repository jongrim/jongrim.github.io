---
layout: post
title: Project Teardown - WarGames Hangman
tags: javascript code project
---
>*This is a lengthy look at the design process of this project. If you want an abbreviated version, check out the [project page]({{ site.url }}/projects/wargames-hangman).*

This week I completed the third assignment of the bootcamp - write a themed hangman game in JavaScript. Having a love for old, cheesy movies where the teenage geek is the hero, I of course chose the classic [WarGames](http://www.imdb.com/title/tt0086567/) for my theme. The idea that a computer can go from playing itself in checkers to deciding that nuclear war is a bad idea is just A+ cinematography in my book.

While hangman is a simple enough game, I had a lot of fun writing this program, and it gave me a chance to dive deeper into JavaScript classes and modules. I wanted to do a walkthrough on how I built out the program, from walking through my mental process of designing the classes to how I figured out how to complete some of the recurring processes of the game. I'm going to walk through the rough order of how I wrote things and why I did things that way. My intended audience for this piece is beginners and those new to object-oriented design, so if you're more experienced, you may want to move on to something else.

>Quick note: I'm not claiming to be an expert at object oriented programming. I think I have a good grasp of many of its fundamental concepts, and I've now written OOP code in 3 languages, but I'm still learning myself, so I'm not guaranteeing this is 100% perfect or that others might not disagree.

# Step One: Pseudocode
First step is always some amount of pseudocoding to make sure that I understand the domain I'm working with. If I don't understand the problem or what the solution should look like, there's no way I'm going to create code that gets it right.

So thinking about this from a pseudocoding approach, what are the steps of hangman?

1. A puzzle is picked by the program, which includes some answer that has to be guessed
2. The player guesses a letter
3. The guessed letter is saved, and then for each letter in the answer, we determine has that letter been guessed?
4. The answer is rewritten with any guessed letters now showing
5. This process is repeated until there are no more guesses left, or the users has solved the puzzle

Those are the basic steps to the game. For me, this was enough of an outline to be able to go and start thinking about how to structure the program, but for others it may be helpful to write even more detailed steps.

# Step Two: Pick Out the Nouns

With the game outlined, I wanted to start thinking about the things or ideas I'll be dealing with in the scope of the program. My goal here is to identify what makes sense to be written and stored as an object versus a different data structure like a list or string.

I honestly don't remember if I learned this in school or heard it on a podcast or something, but one way I approach this is thinking about every noun involved with my program. The general technique is to think about each *thing* or *noun* you're going to be dealing with because often those are items that are good candidates to be expressed as objects. During this exploration, it's also good to start thinking about how these things relate to one another. In the case of the hangman game, a few things came to mind:
- There's two major *things* I'm dealing with: the **game**, and the **puzzle** for that game.
- Furthermore, there's a relationship between the game and the puzzle. Whenever I play hangman, each game has exactly one puzzle - so it's a one to one relationship (ignoring that a puzzle could appear in more than one game).
- But a game doesn't just have a puzzle, it also has some extra attributes that I'll want to track such as the letters guessed so far, the number of guesses remaining, and the current state of the solution.

So with that in mind, I felt like I had pretty decent handle on what objects I needed to make for my program - a **game** object, and some **puzzle** objects.

# Step Three: Writing the Puzzles
I decided I'd start first with the puzzle object since I felt like I had a better mental model of it. There were three properties that I thought every puzzle object should have: a **prompt**, a **hint**, and a **solution**. The prompt is a sentence that gives a directive about how to solve this particular puzzle such as "Complete the Sentence", or "Name the Actor". The hint is more specific, and offers some guidance to what the answer might be, and then the answer is whatever the correct answer is.

Alright, enough talking, **let's write some code**.

As I recall, the first few lines of code I wrote were something along the lines of:

```javascript
var Puzzle = function (prompt, hint, answer) {
    this.prompt = prompt;
    this.hint = hint;
    this.answer = answer;
}
    
var puzzles = [
    new Puzzle(
        "Prompt here",
        "Hint here",
        "Answer here"
    ),
    ...
]
```

You'll see I first define a Puzzle constructor function. For those not familiar, a constructor function is sort of JavaScript's approach to defining a class. You can define once what every *instance* of that class will look like. So that means when I later *instantiate* a Puzzle object with the call `new Puzzle("prompt", "hint", "answer")`, it will have the internal structure defined here. That is the foundation of object-oriented programming.

After the Puzzle object, I then define an array of puzzles. This is simply an array to hold a bunch of Puzzle objects so they can be picked from later.

This really isn't that bad, but I noticed an inefficiency pretty quickly. I was making new Puzzle objects that had the same prompt over and over. I hate typing and aim to type things only once, so this was a sign that I needed to make some changes. What I noticed was that I actually had different types of puzzles, each of which had a certain type of prompt.

At this point I started to look into the new classes feature of JavaScript to see how that worked. As I understand it this is basically syntactic sugar within the language, so these could have easily been written using a similar pattern to above.

So after a bit of rewriting, I came up with the following Puzzle classes:
```javascript
class SimplePuzzle {
    constructor(prompt, hint, answer) {
        this._prompt = prompt;
        this._hint = hint;
        this._answer = answer;
    }

    get prompt() {
        return this._prompt;
    }

    get hint() {
        return this._hint;
    }

    get answer() {
        return this._answer;
    }
}

class SentencePuzzle extends SimplePuzzle {
    constructor(hint, answer) {
        super('Complete the sentence / phrase', hint, answer);
    }
}

class CharacterPuzzle extends SimplePuzzle {
    constructor(hint, answer) {
        super('Name the character', hint, answer);
    }
}
```

There are a few more classes that I made, but you get the idea. Essentially, theres a base class that defines the basic attributes that make up a Puzzle class. From there, specific Puzzle classes set the bits that should be similar across instantiations. For instance, every `SentencePuzzle` object should share the prompt "Complete the sentence / phrase" so I hardcode that bit into the constructor (for those not familiar with a constructor, it's just a bit of code that runs automatically when you create an object of that type - it is useful for setting initial values and such).

This is essentially the meat of my Puzzles module. Beyond this I create one other function to retrieve a random puzzle from the puzzle array.

```javascript
exports.getPuzzle = function getPuzzle() {
    return puzzles[(Math.floor(Math.random() * puzzles.length))];
}
```
Note that I'm exporting the `getPuzzle` function using Node's CommonJS style exports. This makes the function available to other modules, but understanding how it works isn't critical for understanding the structure of the program.

So with that, the Puzzle module was pretty much done. Because I wanted to separate out concerns, there was no need to add any more functionality here. The puzzle itself doesn't need to do anything beyond declare itself and exist. Next!

# Step Four: Writing Game.js
After the puzzles module, I decided to move on to the Game module. I knew this would be the core of my program, and this is where I ended up spending the majority of my time.

A few thoughts I had in mind when I started writing:
- The Game object should store all 'state' data. That is, it should store information about the progress of the game such as what letters have been guessed, is the game over, what's the correct string to write to the screen based on the guessed letters so far. Stuff like that.
- In a similar vein, the Game object should have methods for advancing game state. Because the Game object stores all of the information about its current state, it is most suited to determining what should happen after each guess and as the state changes.
- And finally, there's going to be one Game object for each time the game is loaded. This basically means that when a Game object is made, it needs to get a puzzle, it needs to set the correct number of guesses remaining, etc.

I decided to write the Game object in a more traditional fashion, opting for a normal constructor function. The initial properties were easy to set, and basically matched up with what I had brainstormed:

```javascript
function Game () {
    this.puzzle = puzzles.getPuzzle();
    this.puzzlePrompt = this.puzzle.prompt;
    this.puzzleHint = this.puzzle.hint;
    this.puzzleAnswer = this.puzzle.answer;
    this.guessesRemaining = 6;
    this.lettersGuessed = [];
    this.isOver = false;
    this.finalMessages = [];
    this.solution = this.rewriteSolution();
}
```
Note that game.js imports puzzle.js at the top.

There's nothing real surprising here except for `this.solution`. I'll get to the function that sets it in a moment, but the general thought here is that I needed to store the ever-changing string that represents the letters guessed so far, while also displaying 'blanks' (or more literally '_') for each character not guessed. The function `rewriteSolution()` is how that string gets written, and I'll go into its logic shortly.

From here, I add a few methods to the Game object that truly aren't interesting; things like adding a new letter to the `lettersGuessed` property, checking if `guessesRemaining` is 0 or if the solution has been found (hint: I do that with a RegExp that checks for any '_' still in the solution string). I'll skip over those and instead focus on the interesting methods that really solved problems of how to move the game forward.

**Okay, here it is. Here's how I made the string that showed blank _ characters and the letters guessed so far.**

I think this may be the single biggest question others working on this had. This was the big stumbling block that tripped up a lot of people, so I'm going to walk through the solution I came up with to solve this challenge.

Returning to our pseudocode, this part cover steps 3 and 4 of our list:
- The guessed letter is saved, and then for each letter in the answer, we determine has this letter been guessed
- The answer is rewritten with any guessed letter's filled in

So we know that we are going to save each letter guessed, then we're going to look at each letter of the proper answer and check if it has been guessed. If yes, we write the letter to the screen. If no, we write a '_' instead. This general process translated into this code:

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

Recall that since I'm taking an object-oriented approach, it makes sense to make this an object method so that it can easily access all the stored properties of that object. So now that we see the code, let's run through the process one more time.

For each execution of the function, a `result` string is declared which will hold the value that is ultimately printed to the screen. I also set the puzzle answer to a local variable for ease of reference. The for block within the method then does the following:

*For each character of the puzzle answer:*
- Determine if the character is a letter (done using a RegEx check)
- If yes, check if the letter has been guessed by checking the objects `lettersGuessed` property
    - If guessed, write the letter to the output string. If not, write a '_ ' (I use the HTML encoded &nbsp to avoid rendering issues)
- If the character is not a letter, check if it is a space character
    - If so, substitute the proper HTML code to make a space
- Otherwise, just write the character to the string (this means it's some form of punctuation)

This method is run when the object is first created (and thus no letters have been guessed, so it just gives us a line of blanks), and then again after every guess that is made.

The last important method to cover is the actual `makeGuess()` method:

```javascript
Game.prototype.makeGuess = function (letter) {
    // check if the letter has already been guessed
    if (this.lettersGuessed.indexOf(letter.toUpperCase()) > -1) {
        return;
    }

    // add letter to the guessed letters
    this.addLetter(letter);

    if (this.wordIncludes(this.puzzleAnswer, letter)) {
        // update solution string
        this.solution = this.rewriteSolution();
    } else {
        // decrease remaining guesses
        this.guessesRemaining--;
    }

    // check game status
    this.checkGameStatus();
    if (this.isOver) {
        this.setFinalMessage();
    }

}
```

This method stitches together all of the other Game methods needed to evaluate a guess from the player. I wanted to keep it as simple and readable as possible, so I moved anything that took up more than a line outside of it. This way, all that's left is the game flow logic, and the other helper methods can do specific jobs tasks, such as rewriting the solution like we saw above. The game's `makeGuess()` method is what is called by the `hangman.js` file for every game turn, so let's move on to that file.

# Step Five: Piecing it Together and Writing the User Interaction Bits
Let's quickly recap. At this point, I had written the puzzles that would be used in a game, and the game object itself, which includes the logic needed to do things like make a guess and determine when a game is over. All that is left at this point is writing the user interaction pieces that will accept input and write output back to the player. That's the goal of the `hangman.js` file. I wanted it to contain as little game logic as puzzle, and instead to only be concerned with how to read and write parts. Then, when necessary, it will pass the game logic pieces on to the game object.

I'm going to go through the `hangman.js` file pretty linearly, because that's mostly how it executes.
```javascript
var helpers = require('./helpers.js');
var gameMaker = require('./game.js');

// DOM elements
const body = document.body;
const intro1 = document.querySelector('#intro1');
const intro2 = document.querySelector('#intro2');
const header = document.querySelector('#header');
const prompt = document.querySelector('#prompt');
const hint = document.querySelector('#hint');
const solution = document.querySelector('#solution');
const guesses = document.querySelector('#guesses');
const letters = document.querySelector('#letters');
const finalMessage1 = document.querySelector('#finalMessage1');
const finalMessage2 = document.querySelector('#finalMessage2');
const playAgain = document.querySelector('#playAgain');

const introLine = "Greetings Professor Falken.";
const introLine2 = "Shall we play a game?";
helpers.consoleWriter(introLine, intro1).then(function () {
    helpers.consoleWriter(introLine2, intro2);
});

body.addEventListener('click', loadGame);
body.addEventListener('keydown', loadGame);
```
The first two lines just import our other JS modules. I'm not going to look at the helpers file because it's not real important to the overall game. It just has a couple of functions I wrote to enable some extra stuff that I wanted.

After that, I have a group of lines that do all of my DOM querying. I like having it all organized at the top because it makes it easy to find and read later.

Next, there's a few lines with the helpers bit again. It's purpose is to write the intro lines out to the screen. I encourage you to check it out [live](https://jongrim.github.io/wargames-hangman) to see what it does.

And last, I add a couple of event listeners to know when the player is ready to load the game.


```javascript
function loadGame() {
    let game = new gameMaker.Game()
    
    function writeStats() {
        // updates the changing game stats
        solution.innerHTML = helpers.createElement(game.solution, 'p');
        guesses.innerHTML = helpers.createElement(
            `World destruction in: ${game.guessesRemaining}`, 'p'
        );
        letters.innerHTML = helpers.createElement(
            game.lettersGuessed.join(' '), 'p'
        );
    }

    // nextTurn function executes each turn of the game
    function nextTurn(event) {
        if (/^[a-z]{1}$/.test(event.key)) {
            game.makeGuess(event.key);
            writeStats();
            if (game.isOver) {
                endGame();
            }
        }
    }

    function endGame() {
        // clear the message divs in case there was writing happening unseen
        finalMessage1.innerHTML = '';
        finalMessage1.style.display = 'block';
        finalMessage2.innerHTML = '';
        finalMessage2.style.display = 'block';
        playAgain.innerHTML = '';
        playAgain.style.display = 'block';
        let finalMessages = game.finalMessages
        helpers.consoleWriter(finalMessages[0], finalMessage1)
            .then(function () {
                helpers.consoleWriter(finalMessages[1], finalMessage2)
                    .then(function () {
                        helpers.consoleWriter("Press any key to play again.",
                            playAgain);
                    });
            });
        body.removeEventListener('keydown', nextTurn);
        body.addEventListener('keydown', loadGame);
        
    }

    // remove prior event listeners to avoid loading the game over again
    body.removeEventListener('click', loadGame);
    body.removeEventListener('keydown', loadGame);

    // new event listener to execute game turns
    body.addEventListener('keydown', nextTurn);

    // make sure the end game divs are clear
    finalMessage1.innerHTML = '';
    finalMessage1.style.display = 'none';
    finalMessage2.innerHTML = '';
    finalMessage2.style.display = 'none';
    playAgain.innerHTML = '';
    playAgain.style.display = 'none';
    
    // set initial DOM values
    intro1.style.display = 'none';
    intro2.style.display = 'none';
    header.innerHTML = helpers.createElement('WarGames Hangman', 'h1');
    prompt.innerHTML = helpers.createElement(game.puzzlePrompt, 'p');
    hint.innerHTML = helpers.createElement(game.puzzleHint, 'p');
    writeStats();
}
```
A quick glance through shows that most of what's happening is just clearing or writing HTML elements. The Game object is instantiated at the very top of the function, and then I declare three functions that are used at different times. Skipping over those for the moment, you can then see that I remove the old event listeners, attach a new one to use the `nextTurn` function, and then clear and create some more HTML. Last, the `writeStats()` function is called. This function (look back towards the top for its innards) keeps the game stats up to date.

The `nextTurn()` function is what's called with every key press by the player. It does an initial check that they key is a letter - we don't want to make a guess on a number, punctuation, or shift key - and then it uses the game object to make a guess, rewriting the stats again after. Lastly, it checks if the game is over, and if so, calls the `endGame()` function. That function does more HTML stuff and finally adds new event listeners to allow the game to be restarted.

All in all, the hangman file is pretty boring because it doesn't have to do much that interesting. It just has to accept input and write output, just as I wanted.

# Final Step: Write Really Long Project Write-Up
This was a lengthy look into what is a simple program once you break it down. I wanted to do this though to show some of the thought that can still go into creating a program, even when it's fairly small. Practicing these techniques early on is what I expect will enable me to continue the practice with larger, more complex programs. I also hope, that this could be informative for others who maybe haven't done object-oriented design, or who didn't think about how you can still separate concerns, even in hangman.

---
*Did you like this? Did I get something totally wrong? I'd love to hear from you. Message me @ [jonjongrim@gmail.com](mailto:jonjongrim@gmail.com).*
