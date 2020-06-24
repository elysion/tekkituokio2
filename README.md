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

> Ramda: Lenses, also known as functional references, are a powerful way of looking at, constructing, and using functions on complex data types.

> partial.lenses: Optics decouple the operation to perform on element(s) of a data structure from the details of selecting the element(s) and the details of maintaining data structure invariants.
> In other words, a selection algorithm and data structure invariant maintenance can be expressed as a composition of optics and used with many different operations.

Advantages:
* Declarative
* Composition
* Immutability

Basic operations: view, set, over

---

## View

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

const workPhoneLens = R.lensPath(['phone', 'work'])

// operation(optic,         data)
R.view      (workPhoneLens, person)
// Equivalent to person.phone.work
// Simple!

Output:
{
  area: '+358', 
  number: '...'
}
```

---

## Set

```javascript
// Composition of optics
const workPhoneNumberLens = R.compose(workPhoneLens, R.lensProp('number'))

// operation(optic,               ...  , data)
R.set       (workPhoneNumberLens, '123', person)
// Equivalent to R.evolve({phone: R.evolve({work: R.assoc('number', '123')})})
// Complex! The selection of focus cannot be shared between get and set!

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

## Over

R.over is a modify operation that operates on the focus of the given lens.

```javascript
const person = {
  age: 18
}

const ageLens = R.lensProp('age')
// operation(optic,   ...       , data)
R.over      (ageLens, a => a + 1, person)

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

---

## Partiality

Why *partial*.lenses?

Simply put:
> Reading through the optic gives undefined and writing through the optic replaces the focus with the written value.

Partial behavious means:
* View is undefined if non-existent
* View is undefined if type-mismatch
* Setting to undefined removes element
* Writing an empty object or array produces undefined
* Setting can change type
* Propagating removal

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

Operations, "combinators", lenses, transforms and isomorphisms listed separately in documentation
* Operations: L.get, L.set, L.modify, L.remove, L.removeAll, L.collect
* Optics: 
  * Lenses: ['a', 'b'] (single focus)
  * Traversals: ['a', 'b', L.elems] (multiple focuses)
  * Isomorphisms (like lenses, but inversable)
* Combinators: L.compose, L.flat, L.lazy, L.query, L.ifElse, ...
* Transforms: L.transform, L.*Op

Shorthands:

```javascript
L.compose(a, b, ...) => [a, b, ...]
L.prop('...') => '...'
```

Usage:

```
operation(optic,          ...,    data)             // result
L.get    ('a',                    {a: 1})           // 1
L.collect([L.elems, 'a'],         [{a: 1}, {a: 2}]) // [1, 2]
L.modify ('a',            a => 2, {a: 1})           // {a: 2}
```

```javascript
const L = require('partial.lenses')

const lens = ['a', 'b']

const data = {
  a: {
    b: [1, 2, 3]
  }
}

L.get(lens, data)
// [1, 2, 3]
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

# Examples

```javascript
const anonymizer = anonymizedValues =>
  L.modifyOp((value, key) => (R.has(key, anonymizedValues) ? anonymizedValues[key] : value))
```

L.modify L.query
https://github.com/digabi/registry/blob/f5887502d70503e83fd7143405cc41ea2db53986/server/util/setup-student.js#L842
```javascript
export const bumpExaminations = (student, count = 0) =>
  L.modify(L.query(L.matches(/^\d{4}[K|S]$/)), examinationCode => bumpCode(examinationCode, count), student)
```

https://github.com/digabi/registry/blob/f5887502d70503e83fd7143405cc41ea2db53986/server/util/setup-student.js#L566
```javascript
export const withExamArpaIds = L.transform([
  L.elems,
  registrationsByExaminationLens,
  L.elems,
  'exams',
  L.elems,
  L.when(R.prop('isDigiExam')),
  L.choose(({ examinationcode, ytlRegCode }) => {
    const arpaId = getExamArpaId(examinationcode, ytlRegCode)
    return L.modifyOp(R.assoc('examArpaId', arpaId))
  })
])
```

```javascript
L.transformAsync with L.modifyOp
https://github.com/digabi/registry/blob/176ab2226f2a80277220300a9ee14cb6c845784b/public/js/studentsearch.js#L1012
    const mergeHeldExams = L.transformAsync(
      L.flat(
        'studentExaminations',
        'registrationsByExamination',
        'exams',
        L.choose(({ ytlRegCode, examinationcode }) => [
          'heldExams',
          L.modifyOp(getHeldExams(studentUuid, ytlRegCode, examinationcode))
        ])
      )
    )
    

