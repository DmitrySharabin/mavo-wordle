# Wordle Game

Clone of the famous [Wordle game](https://www.powerlanguage.co.uk/wordle/) built with Mavo.

## Notes for Mavo developers (and “future me” 😜)

### Keyboard coloring

1. We begin coloring the keyboard after the first guess is committed: `attempt > 1`.

2. To be colored, the `key` has to be among characters of any previous guess/attempt:

```js
contains(join(attempts), key)
```

1. If the `key` is among previously *correctly guessed letters*, it's marked as `correct` (and colored green). Else if the *hidden word* contains the `key`, the `key` is marked as `elsewhere` (and colored yellow). Otherwise, it's marked as `absent` (and colored dark gray):

```js
if(key in correctLetters, correct, if(contains(hiddenWord, key), elsewhere, absent))
```

4. 1 + 2 + 3:

```js
if(attempt > 1, if(contains(join(attempts), key), if(key in correctLetters, correct, if(contains(hiddenWord, key), elsewhere, absent))))
```

### How to get a collection of correctly guessed letters?

1. We add letters to the `correctLetters` collection only when the user submitted *a valid guess*: i.e., the guess is *5 characters long* and is *in the list of supported words* — and there were *not more than 6 guesses/attempts*:

```js
guessLength = 5 and attempt <= 6 and guessValid
```

2. We need to filter out letters of the `guessLetters` collection that are *in the correct places inside the hidden word* by splitting the hidden word into letters and matching them with the corresponding letters in a guess:

```js
condense(filter(guessLetters, split(hiddenWord, '') = guessLetters))
```
