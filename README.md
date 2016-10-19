# Tekkituokio 2016-10-18

---

# Agenda

1. Ramda
2. Lenses in general
3. Lenses in Ramda
4. Partial.lenses

---

## Why ramda? (vs. _)

* Currying
 * Every function is curried -> No need for `_.partial`
* Function signatures suit functional programming better
 * Easier to create reusable functions
 * Better composition
* No need for wrapping (i.e. `_.chain`, `_() or .value()`)
* Immutable
 * `R.merge` vs. `_.assign`

```javascript
const filterFalsy = R.filter(R.identity)
const filterFalsy = _.partial(_.filter, _, _.identity)
```

```javascript
const sortDescending = comparator => R.sort(R.flip(comparator))
const getNFirst = n => R.invoker(2, 'slice')(0, n)
const maxScoreForSection = R.pipe(R.prop('questions'), R.pluck('maxScore'), R.sum)
```

> Hey Underscore, You're Doing It Wrong!
https://www.youtube.com/watch?v=m3svKOdZijA
  
---

## Lenses

> Lenses, also known as functional references, are a powerful way of looking at, constructing, and using functions on complex data types.

Advantages:
* Declarative
* Composition
* Immutability

Basic operations: view, set, over (i.e. map (or modify))

---

## View (aka get)

```javascript
const person = {
  name: {
   first: '...',
   last: '...'
  },
  phone: {
    work: {
      area: '+358',
      number: '...'
    }
  }
}

const workPhoneLens = R.compose(R.lensPath(['phone', 'work']))

R.view(workPhoneLens, person)

Output:
{
  area: '+358', 
  number: '...'
}
```

---

## Set

```javascript
const workPhoneNumberLens = R.compose(workPhoneLens, R.lensProp('number'))
R.set(workPhoneNumberLens, '123', person)

Output:
{
  name: {...},
  phone: {
    work: {
      area: '+358',
      number: '123'
    }
...
 
```

Returns a copy of the provided data with the property pointed to by the lens set to the given value. (i.e. does not mutate)

---

## Over (aka modify / map)

```javascript
const person = {
  age: 18
}

const ageLens = R.lensProp('age')
R.over(ageLens, a => a + 1, person)

Output:
{
  age: 19
}
```

---

## partial.lenses

* Part of calmm-js:
https://github.com/calmm-js/documentation/blob/master/introduction-to-calmm.md

* Used for decomposing the application state

* What's the difference between partial lenses and lenses?
 * Optional values

```haskell
type Lens s a = (s -> a, a -> s -> s)
type PLens s a = (Maybe s -> Maybe a, Maybe a -> Maybe s -> Maybe s)
```

---

## Ramda vs. partial.lenses

* `partial.lenses` has a more robust and versatile API

Ramda not partial in all cases:
```javascript
R.set(R.lensPath(["x", "y"]), 1, {})
// { x: { y: 1 } }
R.set(R.compose(R.lensProp("x"), R.lensProp("y")), 1, {})
// TypeError: Cannot read property 'y' of undefined

L.set(P('x', 'y'), 1, {})
// { x: { y: 1 } }
```

Operations and combinators in Ramda:
```javascript
R.view, R.set, R.over
R.lens, R.lensProp, R.lensPath, R.lensIndex
```

Operations and combinators in partial.lenses:
```javascript
L.get, L.set, L.modify, L.remove, L.removeAll, L.collect
L.lens, L.prop, L.props
L.append, L.augument, L.chain, L.choose, L.choice, L.defaults ...
```

---

## Partial.lenses API

* Operations and combinators listed separately in documentation

* Shorthands:

```javascript
L.compose(...) => P(...)
L.prop('...') => '...'
```

Usage:

```javascript
const L = require('partial.lenses')
const P = L.default

const lens = P('a', 'b', L.sequence, ...)

const data = {
  a: {
    b: [...]
  }
}
```

---

## Collect

```javascript
const data = {
  sections: [
    {
      questions: [
        {
          maxScore: 10
        }
      ]
    }
  ]
}

const maxScoreLens = 
  P('sections', L.sequence, 'questions', L.sequence, 'maxScore')
const sumOfMaxScores = R.pipe(L.collect(maxScoreLens), R.sum)(data)
```
