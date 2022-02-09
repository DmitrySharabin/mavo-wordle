# Wordle Game

Clone of the famous [Wordle game](https://www.powerlanguage.co.uk/wordle/) built with Mavo.

## Notes for Mavo developers (and “future me” 😜)

### Guesses

#### How to determine which letter corresponds to which guess (1st, 2nd, etc.)?

The incomplete quotient of a letter `index` (inside the collection of used letters) and `5` (the length of a guess) is the answer: `floor($index / 5)`. E.g., a letter with `$index = 13` belongs to the third guess (guesses are numbered starting from 0) since `floor(13 / 5) = 2`.

#### Coloring

1. We begin coloring letters after the first guess is committed (from the second attempt), i.e., the corresponding letters belong *to the previous guesses*: `row < attempt - 1`, where `row = floor($index / 5)`.

2. If a letter matches the corresponding letter of the hidden word, it's marked as `correct` (and colored green): `get(hiddenWordLetters, $index mod 5) = letter`.

3. A letter that is among letters of the hidden word is marked as `elsewhere` (and colored yellow) unless *it's only one in the hidden word* (i) and is *already in the right spot* in the corresponding guess (ii):

   1. `count(filter(hiddenWordLetters, hiddenWordLetters != letter)) = 4`
   2. `letter in filter(hiddenWordLetters, hiddenWordLetters = thisRowLetters)`

4. In all other cases, a letter is marked as `absent` (and colored dark gray).

5. 1 + 2 + 3 + 4:

```js
if(row < attempt - 1, if(get(hiddenWordLetters, $index mod 5) = letter, correct, if(letter in hiddenWordLetters, if(letter in filter(hiddenWordLetters, hiddenWordLetters = thisRowLetters) and count(filter(hiddenWordLetters, hiddenWordLetters != letter)) = 4, absent, elsewhere), absent)))
```

### Keyboard

#### The `⌫` button

By hitting the Delete button, we expect to delete letters only from *the current guess*, not the previous ones:

```js
deleteif(guessLength > 0, last(game.usedLetters))
```

#### Buttons with letters

We allow constructing guesses not more than 5 characters long:

```js
if(guessLength < 5, add(group('letter': key), usedLetters) & add(key, guessLetters))
```

#### Coloring

1. We begin coloring the keyboard after the first guess is committed: `attempt > 1`.

2. To be colored, the `key` has to be among characters of any previous guess:

```js
contains(join(guesses), key)
```

3. If the `key` is among previously *correctly guessed letters*, it's marked as `correct` (and colored green). Else if the *hidden word* contains the `key`, the `key` is marked as `elsewhere` (and colored yellow). Otherwise, it's marked as `absent` (and colored dark gray):

```js
if(key in correctLetters, correct, if(contains(hiddenWord, key), elsewhere, absent))
```

4. 1 + 2 + 3:

```js
if(attempt > 1, if(contains(join(guesses), key), if(key in correctLetters, correct, if(contains(hiddenWord, key), elsewhere, absent))))
```

#### How to get a collection of correctly guessed letters?

1. We add letters to the `correctLetters` collection only when the user committed *a valid guess*: i.e., the guess is *5 characters long* and is *in the list of supported words*:

```js
guessLength = 5 and guessValid
```

2. We need to filter out letters of the `guessLetters` collection that are *in the correct places inside the hidden word* by splitting the hidden word into letters and matching them with the corresponding letters in a guess:

```js
condense(filter(guessLetters, hiddenWordLetters = guessLetters))
```


### When the game is over?

1. The user wins the game if the guess (5 letters word) matches the hidden word:

```js
guessLength = 5 and guess = hiddenWord
```

2. The user loses the game if they have used his last (the 6th) attempt and the guess (valid 5 letters word) doesn't match the hidden word:

```js
guessLength = 5 and attempt = 6 and guessValid and guess != hiddenWord
```
