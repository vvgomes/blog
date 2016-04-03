---
title: One Line If
layout: post
date: 2010-10-06 22:48
tag:
- javascript
blog: true
---

I believe that every block inside an `if/else` statement should have one line length. Even if you have a simple three lines block logic, it could be named and extracted to a helper method, in order to improve readability and cleanness.

The more conditional blocks an algorithm has, the greater is its [cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity), which means the number of possible paths that algorithm can follow. Then, when you’re reading and understanding a piece of code, everytime you face an `if` statement, you have to save the current flow in your mental stack and then start to read the conditional block till its end in order to figure out what that block is about. The bad news is that, as my friend Filipe Sabella says, unfortunately, you can store no more than **seven** things in your mental stack. Let’s take a look at the following code:

{% highlight javascript %}
function guess(letter) {
  if(hiddenWord.reveal(letter)) {
    hiddenWord.render();
    if(hiddenWord.isEverythingRevealed()) {
      alert('You win!');
      nextLevel();
    }
  }
  else {
    image.showNext();
    if(--chances == 0) {
      alert('You lose!');
      gameOver();
    }
  }
}
{% endhighlight %}

The above Javascript function implements a guessing procedure in a Hangman game. You probably didn’t figure it out before to finish reading the entire code. That’s because you had to understand the **implementation** details in order to figure out the **intention** behind each conditional block. In the end of this process, you have probably named those blocks implicitly in your mind.

Now, let’s take the more external `if/else` blocks and give them names according to the intention behind them. Thinking about the Hangman game, when the player guess a letter, two things can happen:

If the hidden word has that letter, he did a **hit**;
Otherwise, he did a **mistake**.
So, let’s say the same thing using Javascript language instead of English:

{% highlight javascript %}
function guess(letter) {
  if(hiddenWord.reveal(letter))
    hit();
  else
    mistake();
}

function hit() {
  hiddenWord.render();
  if(hiddenWord.isEverythingRevealed()) {
    alert('You win!');
    nextLevel();
  }
}

function mistake() {
  image.showNext();
  if(--chances == 0) {
    alert('You lose!');
    gameOver();
  }
}
{% endhighlight %}

The new version of the `guess` function has explicit names for each alternative computation. We can now read and understand it more quickly, in a single flow, without to store things in our mental stack. Observe that in this new version, the reader get the intention before to enter in the implementation details (actually, we don’t even need to know the implementation at this point).

We can go further and use the Javascript ternary conditional operator to make the `guess` function more straightforward.

{% highlight javascript %}
function guess(letter) {
  hiddenWord.reveal(letter) ? hit() : mistake();
}
{% endhighlight %}

Pretty simple, don’t you think?

We can also keep going on this process and apply the same technique on the `hit` and `mistake` functions. So, let’s take the conditional blocks inside these functions and think about their underlying intentions in order to be able to give them proper names:

The `hit` function has a conditional block describing the behaviour the game should follow when the player **wins** the match;
The `mistake` function has a conditional block describing the behaviour the game should follow when the player has no more chances and then **loses** the game.
Using those names to represent the conditional blocks, we gonna achieve the following result:

{% highlight javascript %}
function guess(letter) {
  hiddenWord.reveal(letter) ? hit() : mistake();
}

function hit() {
  hiddenWord.render();
  if(hiddenWord.isEverythingRevealed())
    win();
}

function mistake() {
  image.showNext();
  if(--chances == 0)
    lose();
}

function win() {
  alert('You win!');
  nextLevel();
}

function lose() {
  alert('You lose!');
  gameOver();
}
{% endhighlight %}

Now, as you can see, the whole code has only one-line `if`s, and the intention behind each conditional path is explicitly named. As well as giving a better reading, we have improved the maintenance by giving a very clear place for each chunk of work. For instance, if you need to change the win behaviour, you just go directly to the win function and do your changes.

## Conclusion

I believe that by using one-line `if`s you can make your code more [literate](https://en.wikipedia.org/wiki/Literate_programming) and descriptive, allowing readers get the overall picture at first and get deeper in details only if they need. Compare the above version of the whole code with the first single `guess` function. Note that in the last version the reading flows more naturally, as if the code was telling you a story.

Moreover, this approach makes software development cheaper and programmers more productive. Given that [the major cost in software development is the maintenance phase](http://amazon.com/dp/0321413091), reading and understanding code in a more effective way could give you significative financial improvements.


Finally, an additional award one-line `if`s give us which I love is, for languages based on C syntax (like Java, Javascript, and C#), you don’t need to use curly braces. As well as helping to reduce the syntax noise (and give your code a Python flavour), it also makes the code more aesthetic and comfortable to the reader eyes.

*I recently presented this topic to my team, check out the slides:*

<script async class="speakerdeck-embed" data-id="261b7a60e37f0131ca5a36d571e0d78c" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>

