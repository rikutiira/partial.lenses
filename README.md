[ [≡](#contents) | [Tutorial](#tutorial) | [Reference](#reference) | [Examples](#examples) | [Background](#background) | [GitHub](https://github.com/calmm-js/partial.lenses) | [Try Lenses!](http://calmm-js.github.io/partial.lenses/) ]

# Partial Lenses

Lenses are basically a [bidirectional](#L-get) [composable](#L-compose)
abstraction for [updating](#L-modify) [selected](#L-choose) elements of
immutable data structures that admits [efficient](#performance) implementation.
This library provides a collection of
*partial* [isomorphisms](#isomorphisms), [lenses](#lenses),
and [traversals](#traversals), collectively known as [optics](#optics), for
manipulating [JSON](http://json.org/) and users can [write new optics](#L-lens)
for manipulating non-JSON objects, such as [Immutable.js](#interfacing)
collections.  A partial lens can *view* optional data, *insert* new data,
*update* existing data and *remove* existing data and can, for example, provide
*defaults* and maintain *required* data structure
parts.  [Try Lenses!](http://calmm-js.github.io/partial.lenses/)

[![npm version](https://badge.fury.io/js/partial.lenses.svg)](http://badge.fury.io/js/partial.lenses) [![Gitter](https://img.shields.io/gitter/room/calmm-js/chat.js.svg)](https://gitter.im/calmm-js/chat) [![Build Status](https://travis-ci.org/calmm-js/partial.lenses.svg?branch=master)](https://travis-ci.org/calmm-js/partial.lenses) [![](https://david-dm.org/calmm-js/partial.lenses.svg)](https://david-dm.org/calmm-js/partial.lenses) [![](https://david-dm.org/calmm-js/partial.lenses/dev-status.svg)](https://david-dm.org/calmm-js/partial.lenses?type=dev)

## Contents

* [Tutorial](#tutorial)
* [Reference](#reference)
  * [Optics](#optics)
    * [Operations on optics](#operations-on-optics)
      * [`L.modify(optic, maybeValue => maybeValue, maybeData)`](#L-modify "L.modify: POptic s a -> (Maybe a -> Maybe a) -> Maybe s -> Maybe s")
      * [`L.remove(optic, maybeData)`](#L-remove "L.remove: POptic s a -> Maybe s -> Maybe s")
      * [`L.set(optic, maybeValue, maybeData)`](#L-set "L.set: POptic s a -> Maybe a -> Maybe s -> Maybe s")
    * [Nesting](#nesting)
      * [`L.compose(...optics)`](#L-compose "L.compose: (POptic s s1, ...POptic sN a) -> POptic s a")
    * [Querying](#querying)
      * [`L.chain(value => optic, optic)`](#L-chain "L.chain: (a -> POptic s b) -> POptic s a -> POptic s b")
      * [`L.choice(...lenses)`](#L-choice "L.choice: (...PLens s a) -> POptic s a")
      * [`L.choose(maybeValue => optic)`](#L-choose "L.choose: (Maybe s -> POptic s a) -> POptic s a")
      * [`L.when(maybeValue => testable)`](#L-when "L.when: (Maybe a -> Boolean) -> POptic a a")
      * [`L.zero`](#L-zero "L.zero: POptic s a")
    * [Recursing](#recursing)
      * [`L.lazy(optic => optic)`](#L-lazy "L.lazy: POptic s a -> POptic s a")
    * [Debugging](#debugging)
      * [`L.log(...labels)`](#L-log "L.log: (...Any) -> POptic s s")
  * [Traversals](#traversals)
    * [Operations on traversals](#operations-on-traversals)
      * [`L.collect(traversal, maybeData)`](#L-collect "L.collect: PTraversal s a -> Maybe s -> [a]")
      * [`L.foldMapOf({empty: () => value, concat: (value, value) => value}, traversal, maybeValue => value, maybeData)`](#L-foldMapOf "L.foldMapOf: {empty: () -> r, concat: (r, r) -> r} -> PTraversal s a -> (Maybe a -> r) -> Maybe s -> r")
    * [Creating new traversals](#creating-new-traversals)
      * [`L.branch({prop: traversal, ...props})`](#L-branch "L.branch: {p1: PTraversal s a, ...pts} -> PTraversal s a")
    * [Traversals and combinators](#traversals-and-combinators)
      * [`L.optional`](#L-optional "L.optional: PTraversal a a")
      * [`L.sequence`](#L-sequence "L.sequence: PTraversal [a] a")
  * [Lenses](#lenses)
    * [Operations on lenses](#operations-on-lenses)
      * [`L.get(lens, maybeData)`](#L-get "L.get: PLens s a -> Maybe s -> Maybe a")
    * [Creating new lenses](#creating-new-lenses)
      * [`L.lens(maybeData => maybeValue, (maybeValue, maybeData) => maybeData)`](#L-lens "L.lens: (Maybe s -> Maybe a) -> ((Maybe a, Maybe s) -> Maybe s) -> PLens s a")
    * [Computing derived props](#computing-derived-props)
      * [`L.augment({prop: object => value, ...props})`](#L-augment "L.augment: {p1: o -> a1, ...ps} -> PLens {...o} {...o, p1: a1, ...ps}")
    * [Enforcing invariants](#enforcing-invariants)
      * [`L.defaults(valueIn)`](#L-defaults "L.defaults: s -> PLens s s")
      * [`L.define(value)`](#L-define "L.define: s -> PLens s s")
      * [`L.normalize(value => value)`](#L-normalize "L.normalize: (s -> s) -> PLens s s")
      * [`L.required(valueOut)`](#L-required "L.required: s -> PLens s s")
      * [`L.rewrite(valueOut => valueOut)`](#L-rewrite "L.rewrite: (s -> s) -> PLens s s")
    * [Lensing arrays](#lensing-arrays)
      * [`L.append`](#L-append "L.append: PLens [a] a")
      * [`L.filter(value => testable)`](#L-filter "L.filter: (a -> Boolean) -> PLens [a] [a]")
      * [`L.find(value => testable)`](#L-find "L.find: (a -> Boolean) -> PLens [a] a")
      * [`L.findWith(...lenses)`](#L-findWith "L.findWith: (PLens s s1, ...PLens sN a) -> PLens [s] a")
      * [`L.index(integer)`](#L-index "L.index: Integer -> PLens [a] a")
    * [Lensing objects](#lensing-objects)
      * [`L.prop(propName)`](#L-prop "L.prop: (p: a) -> PLens {p: a, ...ps} a")
      * [`L.props(...propNames)`](#L-props "L.props: (p1: a1, ...ps) -> PLens {p1: a1, ...ps, ...o} {p1: a1, ...ps}")
    * [Providing defaults](#providing-defaults)
      * [`L.valueOr(valueOut)`](#L-valueOr "L.valueOr: s -> PLens s s")
    * [Adapting to data](#adapting-to-data)
      * [`L.orElse(backupLens, primaryLens)`](#L-orElse "L.orElse: (PLens s a, PLens s a) -> PLens s a")
    * [Read-only mapping](#read-only-mapping)
      * [`L.just(maybeValue)`](#L-just "L.just: Maybe a -> PLens s a")
      * [`L.to(maybeValue => maybeValue)`](#L-to "L.to: (a -> b) -> PLens a b")
    * [Transforming data](#transforming-data)
      * [`L.pick({prop: lens, ...props})`](#L-pick "L.pick: {p1: PLens s a1, ...pls} -> PLens s {p1: a1, ...pls}")
      * [`L.replace(maybeValueIn, maybeValueOut)`](#L-replace "L.replace: Maybe s -> Maybe s -> PLens s s")
  * [Isomorphisms](#isomorphisms)
    * [Operations on isomorphisms](#operations-on-isomorphisms)
      * [`L.getInverse(isomorphism, maybeData)`](#L-getInverse "L.getInverse: PIso a b -> Maybe b -> Maybe a")
    * [Creating new isomorphisms](#creating-new-isomorphisms)
      * [`L.iso(maybeData => maybeValue, maybeValue => maybeData)`](#L-iso "L.iso: (Maybe s -> Maybe a) -> (Maybe a -> Maybe s) -> PIso s a")
    * [Isomorphisms and combinators](#isomorphisms-and-combinators)
      * [`L.identity`](#L-identity "L.identity: PIso s s")
      * [`L.inverse(isomorphism)`](#L-inverse "L.inverse: PIso a b -> PIso b a")
* [Examples](#examples)
  * [An array of ids as boolean flags](#an-array-of-ids-as-boolean-flags)
  * [BST as a lens](#bst-as-a-lens)
  * [Interfacing with Immutable.js](#interfacing)
* [Background](#background)
  * [Motivation](#motivation)
  * [Performance](#performance)
  * [Lenses all the way](#lenses-all-the-way)
  * [Related work](#related-work)

## Tutorial

Let's work with the following sample JSON object:

```js
const sampleTexts = {
  contents: [{ language: "en", text: "Title" },
             { language: "sv", text: "Rubrik" }]
}
```

First we import libraries

```jsx
import * as L from "partial.lenses"
import * as R from "ramda"
```

and compose a parameterized lens for accessing texts:

```js
const textIn = language => L.compose(L.prop("contents"),
                                     L.required([]),
                                     L.normalize(R.sortBy(R.prop("language"))),
                                     L.find(R.whereEq({language})),
                                     L.defaults({language}),
                                     L.prop("text"),
                                     L.valueOr(""))
```

Take a moment to read through the above definition line by line.  Each part
either specifies a step in the path to select the desired element or a way in
which the data structure must be treated at that point.  The purpose of
the [`L.prop(...)`](#L-prop) parts is probably obvious.  The other parts we will
mention below.

### Querying data

Thanks to the parameterized search
part, [`L.find(R.whereEq({language}))`](#L-find), of the lens composition, we
can use it to query texts:

```js
L.get(textIn("sv"), sampleTexts)
// 'Rubrik'
```
```js
L.get(textIn("en"), sampleTexts)
// 'Title'
```

Partial lenses can deal with missing data.  If we use the partial lens to query
a text that does not exist, we get the default:

```js
L.get(textIn("fi"), sampleTexts)
// ''
```

We get this value, rather than `undefined`, thanks to the last
part, [`L.valueOr("")`](#L-valueOr), of our lens composition, which ensures that
we get the specified value rather than `null` or `undefined`.  We get the
default even if we query from `undefined`:

```js
L.get(textIn("fi"), undefined)
// ''
```

With partial lenses, `undefined` is the equivalent of empty or non-existent.

### Updating data

As with ordinary lenses, we can use the same lens to update texts:

```js
L.set(textIn("en"), "The title", sampleTexts)
// { contents: [ { language: 'en', text: 'The title' },
//               { language: 'sv', text: 'Rubrik' } ] }
```

### Inserting data

The same partial lens also allows us to insert new texts:

```js
L.set(textIn("fi"), "Otsikko", sampleTexts)
// { contents: [ { language: 'en', text: 'Title' },
//               { language: 'fi', text: 'Otsikko' },
//               { language: 'sv', text: 'Rubrik' } ] }
```

Note the position into which the new text was inserted.  The array of texts is
kept sorted thanks to
the [`L.normalize(R.sortBy(R.prop("language")))`](#L-normalize) part of our
lens.

### Removing data

Finally, we can use the same partial lens to remove texts:

```js
L.set(textIn("sv"), undefined, sampleTexts)
// { contents: [ { language: 'en', text: 'Title' } ] }
```

Note that a single text is actually a part of an object.  The key to having the
whole object vanish, rather than just the `text` property, is
the [`L.defaults({language})`](#L-defaults) part of our lens composition.
A [`L.defaults(valueIn)`](#L-defaults) lens works *symmetrically*.  When set
with `valueIn`, the result is `undefined`, which means that the focus of the
lens is to be removed.

If we remove all of the texts, we get the required value:

```js
R.pipe(L.set(textIn("sv"), undefined),
       L.set(textIn("en"), undefined))(sampleTexts)
// { contents: [] }
```

The `contents` property is not removed thanks to the
[`L.required([])`](#L-required) part of our lens
composition.  [`L.required`](#L-required) is the dual
of [`L.defaults`](#L-defaults).  [`L.defaults`](#L-defaults) replaces
`undefined` values when viewed and [`L.required`](#L-required) replaces
`undefined` values when set.

Note that unless default and required values are explicitly specified as part of
the lens, they will both be `undefined`.

### Exercises

Take out one (or
more)
[`L.required(...)`](#L-required),
[`L.normalize(...)`](#L-normalize), [`L.defaults(...)`](#L-defaults)
or [`L.valueOr(...)`](#L-valueOr) part(s) from the lens composition and try to
predict what happens when you rerun the examples with the modified lens
composition.  Verify your reasoning by actually rerunning the examples.

Replace [`L.defaults(...)`](#L-defaults) with [`L.valueOr(...)`](#L-valueOr) or vice
verse and try to predict what happens when you rerun the examples with the
modified lens composition.  Verify your reasoning by actually rerunning the
examples.

### Shorthands

For clarity, the previous code snippets avoided some of the shorthands that this
library supports.  In particular,
* [`L.compose(...)`](#L-compose) can be abbreviated as an array
  [`[...]`](#L-compose),
* [`L.prop(string)`](#L-prop) can be abbreviated as [`string`](#L-prop), and
* [`L.set(l, undefined, s)`](#L-set) can be abbreviated
  as [`L.remove(l, s)`](#L-remove).

### Systematic decomposition

It is also typical to compose lenses out of short paths following the schema of
the JSON data being manipulated.  Recall the lens from the start of the
example:

```jsx
L.compose(L.prop("contents"),
          L.required([]),
          L.normalize(R.sortBy(R.prop("language"))),
          L.find(R.whereEq({language})),
          L.defaults({language}),
          L.prop("text"),
          L.valueOr(""))
```

Following the structure or schema of the JSON, we could break this into three
separate lenses:
* a lens for accessing the contents of a data object,
* a parameterized lens for querying a content object from contents, and
* a lens for accessing the text of a content object.

Furthermore, we could organize the lenses into an object following the structure
of the JSON:

```js
const Texts = {
  data: {
    contents: ["contents",
               L.required([]),
               L.normalize(R.sortBy(R.prop("language")))]
  },
  contents: {
    contentIn: language => [L.find(R.whereEq({language})),
                            L.defaults({language})]
  },
  content: {
    text: ["text", L.valueOr("")]
  },
  textIn: language => [Texts.data.contents,
                       Texts.contents.contentIn(language),
                       Texts.content.text]
}
```

We can now say:

```js
L.get(Texts.textIn("sv"), sampleTexts)
// 'Rubrik'
```

This style of organizing lenses is overkill for our toy example.  In a more
realistic case the `sampleTexts` object would contain many more properties.
Also, rather than composing a lens, like `Texts.textIn` above, to access a leaf
property from the root of our object, we might actually compose lenses
incrementally as we inspect the model structure.

### Manipulating multiple items

So far we have used a lens to manipulate individual items.  This library also
supports [traversals](#traversals) that compose with lenses and can target
multiple items.  Continuing on the tutorial example, let's define a traversal
that targets all the texts:

```js
const texts = ["contents",
               L.required([]),
               L.sequence,
               L.choose(R.pipe(L.remove("text"), L.defaults)),
               "text"]
```

What makes the above a traversal is the [`L.sequence`](#L-sequence) part.  Once
a traversal is composed with a lens, the whole results is a traversal.  The
other parts of the above composition should already be familiar from previous
examples.  Note the use of [`L.choose`](#L-choose)
with [`L.defaults`](#L-choose).  We'll get back to that shortly.

Now, we can use the above traversal to [`collect`](#L-collect) all the texts:

```js
L.collect(texts, sampleTexts)
// [ 'Title', 'Rubrik' ]
```

More generally, we can [map and fold](#L-foldMapOf) over texts.  For example, we
can compute the length of the longest text:

```js
const Max = {empty: () => 0, concat: Math.max}
L.foldMapOf(Max, texts, R.length, sampleTexts)
// 6
```

Of course, we can also modify texts.  For example, we could uppercase all the
titles:

```js
L.modify(texts, R.toUpper, sampleTexts)
// { contents: [ { language: 'en', text: 'TITLE' },
//               { language: 'sv', text: 'RUBRIK' } ] }
```

We can also set and remove texts.  Recall the `L.choose` and `L.defaults`
combination from the definition of `texts`.  Like with the `textIn` lens, that
part allows us to remove the whole object instead of just the `text` property:

```js
L.remove(texts, sampleTexts)
// { contents: [] }
```

We can also manipulate texts selectively.  For example, we could truncate all the
texts that are longer than 5 characters:

```js
L.modify([texts, L.when(t => t.length > 5)],
         t => t.slice(0, 5) + "...",
         sampleTexts)
// { contents: [ { language: 'en', text: 'Title' },
//               { language: 'sv', text: 'Rubri...' } ] }
```

## Reference

The combinators provided by this library are available as named imports.
Typically one just imports the library as:

```jsx
import * as L from "partial.lenses"
```

### Optics

The abstractions, [traversals](#traversals), [lenses](#lenses),
and [isomorphisms](#isomorphisms), provided by this library are collectively
known as *optics*.  Traversals can target any number of elements.  Lenses are a
restriction of traversals that target a single element.  Isomorphisms are a
restriction of lenses with an inverse.

Some optics libraries provide many more abstractions, such as "optionals",
"prisms" and "folds", to name a few, forming a DAG.  Aside from being
conceptually important, many of those abstractions are not only useful but
required in a statically typed setting where data structures have precise
constraints on their shapes, so to speak, and operations on data structures must
respect those constraints at *all* times.

In a dynamically typed language like JavaScript, the shapes of run-time objects
are naturally *malleable*.  Nothing immediately breaks if a new object is
created as a copy of another object by adding or removing a property, for
example.  We can exploit this to our advantage by considering all optics as
*partial*.  A partial optic, as manifested in this library, may be intended to
operate on data structures of a specific type, such as arrays or objects, but
also accepts the possibility that it may be given any valid JSON object or
`undefined` as input.  When the input does not match the expectation of a
partial lens, the input is treated as being `undefined`.  This allows specific
partial optics, such as the simple [`L.prop`](#L-prop) lens, to be used in a
wider range of situations than corresponding total optics.

#### Operations on optics

##### <a name="L-modify"></a> [≡](#contents) [`L.modify(optic, maybeValue => maybeValue, maybeData)`](#L-modify "L.modify: POptic s a -> (Maybe a -> Maybe a) -> Maybe s -> Maybe s")

`L.modify` allows one to map over the focused element

```js
L.modify(["elems", 0, "x"], R.inc, {elems: [{x: 1, y: 2}, {x: 3, y: 4}]})
// { elems: [ { x: 2, y: 2 }, { x: 3, y: 4 } ] }
```

or, when using a [traversal](#traversals), elements

```js
L.modify(["elems", L.sequence, "x"], R.dec, {elems: [{x: 1, y: 2}, {x: 3, y: 4}]})
// { elems: [ { x: 0, y: 2 }, { x: 2, y: 4 } ] }
```

of a data structure.

##### <a name="L-remove"></a> [≡](#contents) [`L.remove(optic, maybeData)`](#L-remove "L.remove: POptic s a -> Maybe s -> Maybe s")

`L.remove` allows one to remove the focused element

```js
L.remove([0, "x"], [{x: 1}, {x: 2}, {x: 3}])
// [ { x: 2 }, { x: 3 } ]

```

or, when using a [traversal](#traversals), elements

```js
L.remove([L.sequence, "x", L.when(x => x > 1)], [{x: 1}, {x: 2, y: 1}, {x: 3}])
// [ { x: 1 }, { y: 1 } ]
```

from a data structure.

Note that `L.remove(optic, maybeData)` is equivalent
to [`L.seOptic(lens, undefined, maybeData)`](#L-set).  With partial lenses,
setting to `undefined` typically has the effect of removing the focused element.

##### <a name="L-set"></a> [≡](#contents) [`L.set(optic, maybeValue, maybeData)`](#L-set "L.set: POptic s a -> Maybe a -> Maybe s -> Maybe s")

`L.set` allows one to replace the focused element

```js
L.set(["a", 0, "x"], 11, {id: "z"})
// {a: [{x: 11}], id: 'z'}
```

or, when using a [traversal](#traversals), elements

```js
L.set([L.sequence, "x", L.when(x => x > 1)], 1, [{x: 1}, {x: 2, y: 1}, {x: 3}])
// [ { x: 1 }, { x: 1, y: 1 }, { x: 1 } ]
```

of a data structure.

Note that `L.set(lens, maybeValue, maybeData)` is equivalent
to [`L.modify(lens, R.always(maybeValue), maybeData)`](#L-modify).

#### Nesting

##### <a name="L-compose"></a> [≡](#contents) [`L.compose(...optics)`](#L-compose "L.compose: (POptic s s1, ...POptic sN a) -> POptic s a")

`L.compose` performs composition of optics.  The following equations
characterize composition:

```jsx
                  L.compose() = L.identity
                 L.compose(l) = l
L.modify(L.compose(o, ...os)) = R.compose(L.modify(o), ...os.map(L.modify))
   L.get(L.compose(o, ...os)) = R.pipe(L.get(o), ...os.map(L.get))
```

Furthermore, in this library, an array of optics `[...optics]` is treated as a
composition `L.compose(...optics)`.  Using the array notation, the above
equations can be written as:

```jsx
                  [] = L.identity
                 [l] = l
L.modify([o, ...os]) = R.compose(L.modify(o), ...os.map(L.modify))
   L.get([o, ...os]) = R.pipe(L.get(o), ...os.map(L.get))
```

For example:

```js
L.set(["a", 1], "a", {a: ["b", "c"]})
// { a: [ 'b', 'a' ] }
```
```js
L.get(["a", 1], {a: ["b", "c"]})
// 'c'
```

Note that [`R.compose`](http://ramdajs.com/docs/#compose) is not the same as
`L.compose`.

#### Querying

##### <a name="L-chain"></a> [≡](#contents) [`L.chain(value => optic, optic)`](#L-chain "L.chain: (a -> POptic s b) -> POptic s a -> POptic s b")

`L.chain(toOptic, optic)` is equivalent to

```jsx
L.compose(optic, L.choose(maybeValue =>
  maybeValue === undefined
  ? L.zero
  : toOptic(maybeValue)))
```

Note that with the [`L.just`](#L-just), `L.chain`, [`L.choice`](#L-choice)
and [`L.zero`](#L-zero) combinators, one can consider optics as subsuming the
maybe monad.

##### <a name="L-choice"></a> [≡](#contents) [`L.choice(...lenses)`](#L-choice "L.choice: (...PLens s a) -> POptic s a")

`L.choice` returns a partial optic that acts like the first of the given lenses
whose view is not `undefined` on the given data structure.  When the views of
all of the given lenses are `undefined`, the returned lens acts
like [`L.zero`](#L-zero), which is the identity element of `L.choice`.

For example:

```js
L.modify([L.sequence, L.choice("a", "d")], R.inc, [{R: 1}, {a: 1}, {d: 2}])
// [ { R: 1 }, { a: 2 }, { d: 3 } ]
```

##### <a name="L-choose"></a> [≡](#contents) [`L.choose(maybeValue => optic)`](#L-choose "L.choose: (Maybe s -> POptic s a) -> POptic s a")

`L.choose` creates an optic whose operation is determined by the given function
that maps the underlying view, which can be `undefined`, to an optic.  In other
words, the `L.choose` combinator allows an optic to be constructed *after*
examining the data structure being manipulated.

For example, given:

```js
const majorAxis =
  L.choose(({x, y} = {}) => Math.abs(x) < Math.abs(y) ? "y" : "x")
```

we get:

```js
L.get(majorAxis, {x: 1, y: 2})
// 2
```
```js
L.get(majorAxis, {x: -3, y: 1})
// -3
```
```js
L.modify(majorAxis, R.negate, {x: 2, y: -3})
// { x: 2, y: 3 }
```

##### <a name="L-when"></a> [≡](#contents) [`L.when(maybeValue => testable)`](#L-when "L.when: (Maybe a -> Boolean) -> POptic a a")

`L.when` allows one to selectively skip elements within a traversal or to
selectively turn a lens into a read-only lens whose view is `undefined`.

For example:

```js
L.modify([L.sequence, L.when(x => x > 0)], R.negate, [0,-1,2,-3,4])
// [ 0, -1, -2, -3, -4 ]
```

Note that `L.when(p)` is equivalent to `L.choose(x => p(x) ? L.identity :
L.zero)`.

#### Recursing

##### <a name="L-lazy"></a> [≡](#contents) [`L.lazy(optic => optic)`](#L-lazy "L.lazy: POptic s a -> POptic s a")

`L.lazy` can be used to construct optics lazily.  The function given to `L.lazy`
is passed a forwarding proxy to its return value and can also make forward
references to other optics and possibly construct a recursive optic.

For example, here is a traversal that targets all the non-arrays in a data
structure of nested arrays:

```js
const flatten = L.lazy(rec => {
  const nest = [L.sequence, rec]
  return L.choose(x => R.is(Array, x) ? nest : L.identity)
})
```

Note that the above creates a cyclic representation of the traversal.

Now, for example:

```js
L.collect(flatten, [[[1], 2], 3, [4, [[5]], [6]]])
// [ 1, 2, 3, 4, 5, 6 ]
```
```js
L.modify(flatten, x => x+1, [[[1], 2], 3, [4, [[5]], [6]]])
// [ [ [ 2 ], 3 ], 4, [ 5, [ [Object] ], [ 7 ] ] ]
```
```js
L.remove([flatten, L.when(x => 3 <= x && x <= 5)], [[[1], 2], 3, [4, [[5]], [6]]])
// [ [ [ 1 ], 2 ], [ [ 6 ] ] ]
```

#### Debugging

##### <a name="L-log"></a> [≡](#contents) [`L.log(...labels)`](#L-log "L.log: (...Any) -> POptic s s")

`L.log(...labels)` is an identity optic that
outputs
[`console.log`](https://developer.mozilla.org/en-US/docs/Web/API/Console/log)
messages with the given labels
(or
[format in Node.js](https://nodejs.org/api/console.html#console_console_log_data))
when data flows in either direction, `get` or `set`, through the lens.

For example:

```js
L.get(["x", L.log()], {x: 10})
// get 10
// 10
```
```js
L.set(["x", L.log("x")], "11", {x: 10})
// x get 10
// x set 11
// { x: '11' }
```
```js
L.set(["x", L.log("%s x: %j")], "11", {x: 10})
// get x: 10
// set x: "11"
// { x: '11' }
```

#### <a name="L-zero"></a> [≡](#contents) [`L.zero`](#L-zero "L.zero: POptic s a")

`L.zero` is the identity element of [`L.choice`](#L-choice)
and [`L.chain`](L-chain).  As a traversal, `L.zero` is a traversal of no
elements and as a lens, i.e. when used with [`L.get`](#L-get), `L.zero` is a
read-only lens whose view is always `undefined`.

For example:

```js
L.collect([L.sequence,
           L.choose(x => (R.is(Array, x) ? L.sequence :
                          R.is(Object, x) ? "x" :
                          L.zero))],
          [1, {x: 2}, [3,4]])
// [ 2, 3, 4 ]
```

### Traversals

A traversal operates over a collection of focuses that can
be
[collected](#L-collect),
[folded](#L-foldMapOf), [modified](#L-modify), [set](#L-set)
and [removed](#L-remove).

#### Operations on traversals

##### <a name="L-collect"></a> [≡](#contents) [`L.collect(traversal, maybeData)`](#L-collect "L.collect: PTraversal s a -> Maybe s -> [a]")

`L.collect` returns an array of the elements focused on by the given traversal
or lens from a data structure.  Given a lens, there will be 0 or 1 elements in
the returned array.  Note that a partial *lens* always targets an element, but
`L.collect` implicitly skips elements that are `undefined`.  Given a traversal,
there can be any number of elements in the array returned by `L.collect`.

For example:

```js
L.collect(["xs", L.sequence, "x"], {xs: [{x: 1}, {x: 2}]})
// [ 1, 2 ]
```

`L.collect(traversal, maybeData)` is equivalent
to [`L.foldMapOf(Collect, traversal, toCollect, maybeData)`](#L-foldMapOf) where
`Collect` and `toCollect` are defined as follows:

```js
const Collect = {empty: R.always([]), concat: R.concat}
const toCollect = x => x !== undefined ? [x] : []
```

So:

```js
L.foldMapOf(Collect, ["xs", L.sequence, "x"], toCollect, {xs: [{x: 1}, {x: 2}]})
// [ 1, 2 ]
```

The internal implementation of `L.collect` is optimized and faster than the
above naïve implementation.

##### <a name="L-foldMapOf"></a> [≡](#contents) [`L.foldMapOf({empty: () => value, concat: (value, value) => value}, traversal, maybeValue => value, maybeData)`](#L-foldMapOf "L.foldMapOf: {empty: () -> r, concat: (r, r) -> r} -> PTraversal s a -> (Maybe a -> r) -> Maybe s -> r")

`L.foldMapOf({empty, concat}, t, aM2r, s)` performs a map, using given function
`aM2r`, and fold, using the given `concat` and `empty` operations, over the
elements focused on by the given traversal or lens `t` from the given data
structure `s`.  The `concat` operation and the constant returned by `empty()`
should form
a
[monoid](https://github.com/rpominov/static-land/blob/master/docs/spec.md#monoid) over
the values returned by `aM2r`.

For example:

```js
const Sum = {empty: () => 0, concat: (x, y) => x + y}
L.foldMapOf(Sum, L.sequence, x => x, [1,2,3])
// 6
```

#### Creating new traversals

##### <a name="L-branch"></a> [≡](#contents) [`L.branch({prop: traversal, ...props})`](#L-branch "L.branch: {p1: PTraversal s a, ...pts} -> PTraversal s a")

`L.branch` is given a template object of traversals and returns a traversal that
visits all the properties of an object according to the template.

For example:

```js
L.collect(L.branch({first: L.sequence, second: L.identity}), {first: ["x"], second: "y"})
// [ 'x', 'y' ]
```

See [BST traversal](#bst-traversal) for a more meaningful example.

#### Traversals and combinators

##### <a name="L-optional"></a> [≡](#contents) [`L.optional`](#L-optional "L.optional: PTraversal a a")

`L.optional` is a traversal over an optional element.  When the focus of
`L.optional` is `undefined`, the traversal is empty.  Otherwise the traversal is
over the focused element.

As an example, consider the difference between:

```js
L.set([L.sequence, "x"], 3, [{x: 1}, {y: 2}])
// [ { x: 3 }, { y: 2, x: 3 } ]
```

and:

```js
L.set([L.sequence, "x", L.optional], 3, [{x: 1}, {y: 2}])
// [ { x: 3 }, { y: 2 } ]
```

Note that `L.optional` is equivalent to [`L.when(x => x !== undefined)`](#L-when).

##### <a name="L-sequence"></a> [≡](#contents) [`L.sequence`](#L-sequence "L.sequence: PTraversal [a] a")

`L.sequence` is a traversal over an array.

For example:

```js
L.modify(["xs", L.sequence, "x"], R.add(1), {xs: [{x: 1}, {x: 2}]})
// { xs: [ { x: 2 }, { x: 3 } ] }
```

### Lenses

Lenses always have a single focus which can be [viewed](#L-get) directly.

#### Operations on lenses

##### <a name="L-get"></a> [≡](#contents) [`L.get(lens, maybeData)`](#L-get "L.get: PLens s a -> Maybe s -> Maybe a")

`L.get` returns the focused element from a data structure.

For example:

```js
L.get("y", {x: 112, y: 101})
// 101
```

Note that `L.get` does not work on [traversals](#traversals).

#### Creating new lenses

##### <a name="L-lens"></a> [≡](#contents) [`L.lens(maybeData => maybeValue, (maybeValue, maybeData) => maybeData)`](#L-lens "L.lens: (Maybe s -> Maybe a) -> ((Maybe a, Maybe s) -> Maybe s) -> PLens s a")

`L.lens` creates a new primitive lens.  The first parameter is the *getter* and
the second parameter is the *setter*.  The setter takes two parameters: the
first is the value written and the second is the data structure to write into.

One should think twice before introducing a new primitive lens&mdash;most of the
combinators in this library have been introduced to reduce the need to write new
primitive lenses.  With that said, there are still valid reasons to create new
primitive lenses.  For example, here is a lens that we've used in production,
written with the help of [Moment.js](http://momentjs.com/), to bidirectionally
convert a pair of `start` and `end` times to a duration:

```js
const timesAsDuration = L.lens(
  ({start, end} = {}) => {
    if (undefined === start)
      return undefined
    if (undefined === end)
      return "Infinity"
    return moment.duration(moment(end).diff(moment(start))).toJSON()
  },
  (duration, {start = moment().toJSON()} = {}) => {
    if (undefined === duration || "Infinity" === duration) {
      return {start}
    } else {
      return {
        start,
        end: moment(start).add(moment.duration(duration)).toJSON()
      }
    }
  }
)
```

Now, for example:

```js
L.get(timesAsDuration,
      {start: "2016-12-07T09:39:02.451Z",
       end: moment("2016-12-07T09:39:02.451Z").add(10, "hours").toISOString()})
// "PT10H"
```

```js
L.set(timesAsDuration,
      "PT10H",
      {start: "2016-12-07T09:39:02.451Z",
       end: "2016-12-07T09:39:02.451Z"})
// { end: '2016-12-07T19:39:02.451Z',
//   start: '2016-12-07T09:39:02.451Z' }
```

When composed with [`L.pick`](#L-pick), to flexibly pick the `start` and `end`
times, the above can be adapted to work in a wide variety of cases.  However,
the above lens will never be added to this library, because it would require
adding dependency to [Moment.js](http://momentjs.com/).

#### Computing derived props

##### <a name="L-augment"></a> [≡](#contents) [`L.augment({prop: object => value, ...props})`](#L-augment "L.augment: {p1: o -> a1, ...ps} -> PLens {...o} {...o, p1: a1, ...ps}")

`L.augment` is given a template of functions to compute new properties.  When
not viewing or setting a defined object, the result is `undefined`.  When
viewing a defined object, the object is extended with the computed properties.
When set with a defined object, the extended properties are removed.

For example:

```js
L.modify(L.augment({y: r => r.x + 1}), r => ({x: r.x + r.y, y: 2, z: r.x - r.y}), {x: 1})
// { x: 3, z: -1 }
```

#### Enforcing invariants

##### <a name="L-defaults"></a> [≡](#contents) [`L.defaults(valueIn)`](#L-defaults "L.defaults: s -> PLens s s")

`L.defaults` is used to specify a default context or value for an element in
case it is missing.  When set with the default value, the effect is to remove
the element.  This can be useful for both making partial lenses with propagating
removal and for avoiding having to check for and provide default values
elsewhere.

For example:

```js
L.get(["items", L.defaults([])], {})
// []
```
```js
L.get(["items", L.defaults([])], {items: [1, 2, 3]})
// [ 1, 2, 3 ]
```
```js
L.set(["items", L.defaults([])], [], {items: [1, 2, 3]})
// undefined
```

Note that `L.defaults(valueIn)` is equivalent
to [`L.replace(undefined, valueIn)`](#L-replace).

##### <a name="L-define"></a> [≡](#contents) [`L.define(value)`](#L-define "L.define: s -> PLens s s")

`L.define` is used to specify a value to act as both the default value and the
required value for an element.

```js
L.get(["x", L.define(null)], {y: 10})
// null
```
```js
L.set(["x", L.define(null)], undefined, {y: 10})
// { y: 10, x: null }
```

Note that `L.define(value)` is equivalent to `[L.required(value),
L.defaults(value)]`.

##### <a name="L-normalize"></a> [≡](#contents) [`L.normalize(value => value)`](#L-normalize "L.normalize: (s -> s) -> PLens s s")

`L.normalize` maps the value with same given transform when viewed and set and
implicitly maps `undefined` to `undefined`.

One use case for `normalize` is to make it easy to determine whether, after a
change, the data has actually changed.  By keeping the data normalized, a
simple [`R.equals`](http://ramdajs.com/docs/#equals) comparison will do.

##### <a name="L-required"></a> [≡](#contents) [`L.required(valueOut)`](#L-required "L.required: s -> PLens s s")

`L.required` is used to specify that an element is not to be removed; in case it
is removed, the given value will be substituted instead.

For example:

```js
L.remove(["items", 0], {items: [1]})
// undefined
```
```js
L.remove([L.required({}), "items", 0], {items: [1]})
// {}
```
```js
L.remove(["items", L.required([]), 0], {items: [1]})
// { items: [] }
```

Note that `L.required(valueOut)` is equivalent
to [`L.replace(valueOut, undefined)`](#L-replace).

##### <a name="L-rewrite"></a> [≡](#contents) [`L.rewrite(valueOut => valueOut)`](#L-rewrite "L.rewrite: (s -> s) -> PLens s s")

`L.rewrite` maps the value with the given transform when set and implicitly maps
`undefined` to `undefined`.  One use case for `rewrite` is to re-establish data
structure invariants after changes.

#### Lensing arrays

##### <a name="L-append"></a> [≡](#contents) [`L.append`](#L-append "L.append: PLens [a] a")

`L.append` is a write-only lens that can be used to append values to an array.
The view of `L.append` is always `undefined`.

For example:

```js
L.get(L.append, ["x"])
// undefined
```
```js
L.set(L.append, "x", undefined)
// [ 'x' ]
```
```js
L.set(L.append, "x", ["z", "y"])
// [ 'z', 'y', 'x' ]
```

Note that `L.append` is equivalent to [`L.index(i)`](#L-index) with the index
`i` set to the length of the focused array or 0 in case the focus is not a
defined array.

##### <a name="L-filter"></a> [≡](#contents) [`L.filter(value => testable)`](#L-filter "L.filter: (a -> Boolean) -> PLens [a] [a]")

`L.filter` operates on arrays.  When not viewing an array, the result is
`undefined`.  When viewing an array, only elements matching the given predicate
will be returned.  When set, the resulting array will be formed by concatenating
the set array and the complement of the filtered context.  If the resulting
array would be empty, the whole result will be `undefined`.

For example:

```js
L.remove(L.filter(x => x <= 2), [3,1,4,1,5,9,2])
// [ 3, 4, 5, 9 ]
```

*Note:* An alternative design for filter could implement a smarter algorithm to
combine arrays when set.  For example, an algorithm based
on [edit distance](https://en.wikipedia.org/wiki/Edit_distance) could be used to
maintain relative order of elements.  While this would not be difficult to
implement, it doesn't seem to make sense, because in most cases use
of [`L.normalize`](#L-normalize) would be preferable.

##### <a name="L-find"></a> [≡](#contents) [`L.find(value => testable)`](#L-find "L.find: (a -> Boolean) -> PLens [a] a")

`L.find` operates on arrays like [`L.index`](#L-index), but the index to be
viewed is determined by finding the first element from the input array that
matches the given predicate.  When no matching element is found the effect is
same as with [`L.append`](#L-append).

```js
L.remove(L.find(x => x <= 2), [3,1,4,1,5,9,2])
// [ 3, 4, 1, 5, 9, 2 ]
```

##### <a name="L-findWith"></a> [≡](#contents) [`L.findWith(...lenses)`](#L-findWith "L.findWith: (PLens s s1, ...PLens sN a) -> PLens [s] a")

`L.findWith(...lenses)` chooses an index from an array through which the given
lens, [`[...lenses]`](#L-compose), focuses on a defined item and then returns a
lens that focuses on that item.

For example:

```js
L.get(L.findWith("x"), [{z: 6}, {x: 9}, {y: 6}])
// 9
```
```js
L.set(L.findWith("x"), 3, [{z: 6}, {x: 9}, {y: 6}])
// [ { z: 6 }, { x: 3 }, { y: 6 } ]
```

##### <a name="L-index"></a> [≡](#contents) [`L.index(integer)`](#L-index "L.index: Integer -> PLens [a] a")

`L.index(integer)` or just `integer` focuses on the specified array index.

* When not viewing a defined array index, the result is `undefined`.
* When setting to `undefined`, the element is removed from the resulting array,
  shifting all higher indices down by one.  If the result would be an empty
  array, the whole result will be `undefined`.
* When setting a defined value to an index that is higher than the length of the
  array, the missing elements will be filled with `null`.

**NOTE:** There is a gotcha related to removing elements from an array.  Namely,
when the last element is removed, the result is `undefined` rather than an empty
array.  This is by design, because this allows the removal to propagate upwards.
It is not uncommon, however, to have cases where removing the last element from
an array must not remove the array itself.  Consider the following examples
without [`L.required([])`](#L-required):

```js
L.remove(0, ["a", "b"])
// [ 'b' ]
```
```js
L.remove(0, ["b"])
// undefined
```
```js
L.remove(["elems", 0], {elems: ["b"], some: "thing"})
// { some: 'thing' }
```

Then consider the same examples with [`L.required([])`](#L-required):

```js
L.remove([L.required([]), 0], ["a", "b"])
// [ 'b' ]
```
```js
L.remove([L.required([]), 0], ["b"])
// []
```
```js
L.remove(["elems", L.required([]), 0], {elems: ["b"], some: "thing"})
// { elems: [], some: 'thing' }
```

There is a related gotcha with [`L.required`](#L-required).  Consider the
following example:

```js
L.remove(L.required([]), [])
// []
```
```js
L.get(L.required([]), [])
// undefined
```

In other words, [`L.required`](#L-required) works in both directions.  Thanks to
the handling of `undefined` within partial lenses, this is often not a problem,
but sometimes you need the "default" value both ways.  In that case you can
use [`L.define`](#L-define).

#### Lensing objects

##### <a name="L-prop"></a> [≡](#contents) [`L.prop(propName)`](#L-prop "L.prop: (p: a) -> PLens {p: a, ...ps} a")

`L.prop(string)` or `string` focuses on the specified object property.

* When not viewing a defined object property, the result is `undefined`.
* When setting property to `undefined`, the property is removed from the result.
  If the result would be an empty object, the whole result will be `undefined`.

When setting or removing properties, the order of keys is preserved.

For example:

```js
L.get("y", {x: 1, y: 2, z: 3})
// 2
```
```js
L.set("y", -2, {x: 1, y: 2, z: 3})
// { x: 1, y: -2, z: 3 }
```

##### <a name="L-props"></a> [≡](#contents) [`L.props(...propNames)`](#L-props "L.props: (p1: a1, ...ps) -> PLens {p1: a1, ...ps, ...o} {p1: a1, ...ps}")

`L.props` focuses on a subset of properties of an object, allowing one to treat
the subset of properties as a unit.  The view of `L.props` is `undefined` when
none of the properties is defined.  Otherwise the view is an object containing a
subset of the properties.  Setting through `L.props` updates the whole subset of
properties, which means that any missing properties are removed if they did
exists previously.  When set, any extra properties are ignored.

```js
L.set(L.props("x", "y"), {x: 4}, {x: 1, y: 2, z: 3})
// { x: 4, z: 3 }
```

Note that `L.props(k1, ..., kN)` is equivalent to [`L.pick({[k1]: k1, ..., [kN]:
kN})`](#L-pick).

#### Providing defaults

##### <a name="L-valueOr"></a> [≡](#contents) [`L.valueOr(valueOut)`](#L-valueOr "L.valueOr: s -> PLens s s")

`L.valueOr` is an asymmetric lens used to specify a default value in case the
focus is `undefined` or `null`.  When set, `L.valueOr` behaves like the identity
lens.

For example:

```js
L.get(L.valueOr(0), null)
// 0
```
```js
L.set(L.valueOr(0), 0, 1)
// 0
```
```js
L.remove(L.valueOr(0), 1)
// undefined
```

#### Adapting to data

##### <a name="L-orElse"></a> [≡](#contents) [`L.orElse(backupLens, primaryLens)`](#L-orElse "L.orElse: (PLens s a, PLens s a) -> PLens s a")

`L.orElse(backupLens, primaryLens)` acts like `primaryLens` when its view is not
`undefined` and otherwise like `backupLens`.  You can use `L.orElse` on its own
with [`R.reduceRight`](http://ramdajs.com/docs/#reduceRight)
(and [`R.reduce`](http://ramdajs.com/docs/#reduce)) to create an associative
choice over lenses or use `L.orElse` to specify a default or backup lens
for [`L.choice`](#L-choice), for example.

#### Read-only mapping

##### <a name="L-just"></a> [≡](#contents) [`L.just(maybeValue)`](#L-just "L.just: Maybe a -> PLens s a")

`L.just` returns a read-only lens whose view is always the given value.  In
other words, for all `x`, `y` and `z`:

```jsx
   L.get(L.just(z), x) = z
L.set(L.just(z), y, x) = x
```

Note that `L.just(x)` is equivalent to [`L.to(_ => x)`](#L-to).

`L.just` can be seen as the unit function of the monad formed
with [`L.chain`](#L-chain).

##### <a name="L-to"></a> [≡](#contents) [`L.to(maybeValue => maybeValue)`](#L-to "L.to: (a -> b) -> PLens a b")

`L.to` creates a read-only lens whose view is determined by the given function.

For example:

```js
L.get(["x", L.to(x => x + 1)], {x: 1})
// 2
```
```js
L.set(["x", L.to(x => x + 1)], 3, {x: 1})
// { x: 1 }
```

#### Transforming data

##### <a name="L-pick"></a> [≡](#contents) [`L.pick({prop: lens, ...props})`](#L-pick "L.pick: {p1: PLens s a1, ...pls} -> PLens s {p1: a1, ...pls}")

`L.pick` creates a lens out of the given object template of lenses and allows
one to pick apart a data structure and then put it back together.  When viewed,
an object is created, whose properties are obtained by viewing through the
lenses of the template.  When set with an object, the properties of the object
are set to the context via the lenses of the template.  `undefined` is treated
as the equivalent of empty or non-existent in both directions.

For example, let's say we need to deal with data and schema in need of some
semantic restructuring:

```js
const sampleFlat = {px: 1, py: 2, vx: 1.0, vy: 0.0}
```

We can use `L.pick` to create lenses to pick apart the data and put it back
together into a more meaningful structure:

```js
const asVec = prefix => L.pick({x: prefix + "x", y: prefix + "y"})
const sanitize = L.pick({pos: asVec("p"), vel: asVec("v")})
```

We now have a better structured view of the data:

```js
L.get(sanitize, sampleFlat)
// { pos: { x: 1, y: 2 }, vel: { x: 1, y: 0 } }
```

That works in both directions:

```js
L.modify([sanitize, "pos", "x"], R.add(5), sampleFlat)
// { px: 6, py: 2, vx: 1, vy: 0 }
```

**NOTE:** In order for a lens created with `L.pick` to work in a predictable
manner, the given lenses must operate on independent parts of the data
structure.  As a trivial example, in `L.pick({x: "same", y: "same"})` both of
the resulting object properties, `x` and `y`, address the same property of the
underlying object, so writing through the lens will give unpredictable results.

Note that, when set, `L.pick` simply ignores any properties that the given
template doesn't mention.  Also note that the underlying data structure need not
be an object.

##### <a name="L-replace"></a> [≡](#contents) [`L.replace(maybeValueIn, maybeValueOut)`](#L-replace "L.replace: Maybe s -> Maybe s -> PLens s s")

`L.replace(maybeValueIn, maybeValueOut)`, when viewed, replaces the value
`maybeValueIn` with `maybeValueOut` and vice versa when set.

For example:

```js
L.get(L.replace(1, 2), 1)
// 2
```
```js
L.set(L.replace(1, 2), 2, 0)
// 1
```

The main use case for `replace` is to handle optional and required properties
and elements.  In most cases, rather than using `replace`, you will make
selective use of [`defaults`](#L-defaults), [`required`](#L-required)
and [`define`](#L-define).

### Isomorphisms

The focus of an isomorphism is the whole data structure rather than a part of
it.  Furthermore, an isomorphism can be [inverted](#L-inverse).  More
specifically, a lens, `iso`, is an isomorphism iff the following equations hold
for all `x` and `y` in the domain and range, respectively, of the lens:

```jsx
L.set(iso, L.get(iso, x), undefined) = x
L.get(iso, L.set(iso, y, undefined)) = y
```

The above equations mean that `x => L.get(iso, x)` and `y => L.set(iso, y,
undefined)` are inverses of each other.

#### Operations on isomorphisms

##### <a name="L-getInverse"></a> [≡](#contents) [`L.getInverse(isomorphism, maybeData)`](#L-getInverse "L.getInverse: PIso a b -> Maybe b -> Maybe a")

`L.getInverse` views through an isomorphism in the inverse direction.

For example:

```js
const numeric = f => x => typeof x === "number" ? f(x) : undefined
const offBy1 = L.iso(numeric(R.inc), numeric(R.dec))
L.getInverse(offBy1, 1)
// 0
```

Note that `L.getInverse(iso, data)` is equivalent
to [`L.set(iso, data, undefined)`](#L-get).

Also note that, while `L.getInverse` makes most sense when used with an
isomorphism, it is valid to use `L.getInverse` with *partial* lenses in general.
Doing so essentially constructs a minimal data structure that contains the given
value.  For example:

```js
L.getInverse("meaning", 42)
// { meaning: 42 }
```

#### Creating new isomorphisms

##### <a name="L-iso"></a> [≡](#contents) [`L.iso(maybeData => maybeValue, maybeValue => maybeData)`](#L-iso "L.iso: (Maybe s -> Maybe a) -> (Maybe a -> Maybe s) -> PIso s a")

`L.iso` creates a new primitive isomorphism.

For example:

```js
const negate = L.iso(numeric(R.negate), numeric(R.negate))
L.get([negate, L.inverse(negate)], 112)
// 112
```

#### Isomorphisms and combinators

##### <a name="L-identity"></a> [≡](#contents) [`L.identity`](#L-identity "L.identity: PIso s s")

`L.identity` is the identity element of lens composition and also the identity
isomorphism.  The following equations characterize `L.identity`:

```jsx
      L.get(L.identity, x) = x
L.modify(L.identity, f, x) = f(x)
  L.compose(L.identity, l) = l
  L.compose(l, L.identity) = l
```

##### <a name="L-inverse"></a> [≡](#contents) [`L.inverse(isomorphism)`](#L-inverse "L.inverse: PIso a b -> PIso b a")

`L.inverse` returns the inverse of the given isomorphism.  Note that this
operation only makes sense on isomorphisms.

For example:

```js
L.get(L.inverse(offBy1), 1)
// 0
```

## Examples

If you are new to lenses, then you probably want to start with
the [tutorial](#tutorial).

### An array of ids as boolean flags

A case that we have run into multiple times is where we have an array of
constant strings such as

```js
const sampleFlags = ["id-19", "id-76"]
```

that we wish to manipulate as if it was a collection of boolean flags.  Here is
a parameterized lens that does just that:

```js
const flag = id => [L.normalize(R.sortBy(R.identity)),
                    L.find(R.equals(id)),
                    L.replace(undefined, false),
                    L.replace(id, true)]
```

Now we can treat individual constants as boolean flags:

```js
L.get(flag("id-69"), sampleFlags)
// false
```
```js
L.get(flag("id-76"), sampleFlags)
// true
```

In both directions:

```js
L.set(flag("id-69"), true, sampleFlags)
// ['id-19', 'id-69', 'id-76']
```
```js
L.set(flag("id-76"), false, sampleFlags)
// ['id-19']
```

### BST as a lens

Binary search trees might initially seem to be outside the scope of definable
lenses.  However, given basic BST operations, one could easily wrap them as a
primitive partial lens.  But could we leverage lens combinators to build a BST
lens more compositionally?  We can.  The [`L.choose`](#L-choose) combinator
allows for dynamic construction of lenses based on examining the data structure
being manipulated.  Inside [`L.choose`](#L-choose) we can write the ordinary BST
logic to pick the correct branch based on the key in the currently examined node
and the key that we are looking for.  So, here is our first attempt at a BST
lens:

```js
const searchAttempt = key => L.lazy(rec => {
  const smaller = ["smaller", rec]
  const greater = ["greater", rec]
  const found = L.defaults({key})
  return L.choose(n => {
    if (!n || key === n.key)
      return found
    return key < n.key ? smaller : greater
  })
})

const valueOfAttempt = key => [searchAttempt(key), "value"]
```

Note that we also make use of the [`L.lazy`](#L-lazy) combinator to create a
recursive lens with a cyclic representation.

This actually works to a degree.  We can use the `valueOfAttempt` lens
constructor to build a binary tree.  Here is a little helper to build a tree
from pairs:

```js
const fromPairs =
  R.reduce((t, [k, v]) => L.set(valueOfAttempt(k), v, t), undefined)
```

Now:

```js
const sampleBST = fromPairs([[3, "g"], [2, "a"], [1, "m"], [4, "i"], [5, "c"]])
sampleBST
// { key: 3,
//   value: 'g',
//   smaller: { key: 2, value: 'a', smaller: { key: 1, value: 'm' } },
//   greater: { key: 4, value: 'i', greater: { key: 5, value: 'c' } } }
```

However, the above `searchAttempt` lens constructor does not maintain the BST
structure when values are being removed:

```js
L.remove(valueOfAttempt(3), sampleBST)
// { key: 3,
//   smaller: { key: 2, value: 'a', smaller: { key: 1, value: 'm' } },
//   greater: { key: 4, value: 'i', greater: { key: 5, value: 'c' } } }
```

How do we fix this?  We could check and transform the data structure to a BST
after changes.  The [`L.rewrite`](#L-rewrite) combinator can be used for that
purpose.  Here is a naïve rewrite to fix a tree after value removal:

```js
const naiveBST = L.rewrite(n => {
  if (undefined !== n.value) return n
  const s = n.smaller, g = n.greater
  if (!s) return g
  if (!g) return s
  return L.set(search(s.key), s, g)
})
```

Here is a working `search` lens and a `valueOf` lens constructor:

```js
const search = key => L.lazy(rec => {
  const smaller = ["smaller", rec]
  const greater = ["greater", rec]
  const found = L.defaults({key})
  return [naiveBST, L.choose(n => {
    if (!n || key === n.key)
      return found
    return key < n.key ? smaller : greater
  })]
})

const valueOf = key => [search(key), "value"]
```

Now we can also remove values from a binary tree:

```js
L.remove(valueOf(3), sampleBST)
// { key: 4,
//   value: 'i',
//   greater: { key: 5, value: 'c' },
//   smaller: { key: 2, value: 'a', smaller: { key: 1, value: 'm' } } }
```

As an exercise, you could improve the rewrite to better maintain balance.
Perhaps you might even enhance it to maintain a balance condition such
as [AVL](https://en.wikipedia.org/wiki/AVL_tree)
or [Red-Black](https://en.wikipedia.org/wiki/Red%E2%80%93black_tree).  Another
worthy exercise would be to make it so that the empty binary tree is `null`
rather than `undefined`.

#### BST traversal

What about [traversals](#traverals) over BSTs?  We can use
the [`L.branch`](#L-branch) combinator to define an in-order traversal over the
values of a BST:

```js
const values = L.lazy(rec => [
  L.optional,
  naiveBST,
  L.branch({smaller: rec,
            value: L.identity,
            greater: rec})])
```

Given a binary tree `sampleBST` we can now manipulate it as a whole.  For
example:

```js
const Concat = {empty: () => "", concat: R.concat}
L.foldMapOf(Concat, values, R.toUpper, sampleBST)
// 'MAGIC'
```
```js
L.modify(values, R.toUpper, sampleBST)
// { key: 3,
//   value: 'G',
//   smaller: { key: 2, value: 'A', smaller: { key: 1, value: 'M' } },
//   greater: { key: 4, value: 'I', greater: { key: 5, value: 'C' } } }
```
```js
L.remove([values, L.when(x => x > "e")], sampleBST)
// { key: 5, value: 'c', smaller: { key: 2, value: 'a' } }
```

### <a name="interfacing"></a> Interfacing with Immutable.js

[Immutable.js](http://facebook.github.io/immutable-js/) is a popular library
providing immutable data structures.  As argued
in
[Lenses with Immutable.js](https://medium.com/@drboolean/lenses-with-immutable-js-9bda85674780#.kzq41xgw3) it
can be useful to wrap such libraries as [optics](#optics).

When interfacing external libraries with partial lenses one does need to
consider whether and how to support partiality.  Partial lenses allow one to
insert new and remove existing elements rather than just view and update
existing elements.

#### `List` indexing

Here is a primitive partial lens for
indexing [`List`](http://facebook.github.io/immutable-js/docs/#/List) written
using [`L.lens`](#L-lens):

```js
const getList = i => xs => Immutable.List.isList(xs) ? xs.get(i) : undefined

const setList = i => (x, xs) => {
  if (!Immutable.List.isList(xs))
    xs = Immutable.List()
  if (x !== undefined)
    return xs.set(i, x)
  xs = xs.delete(i)
  return xs.size ? xs : undefined
}

const idxList = i => L.lens(getList(i), setList(i))
```

Note how the above uses `isList` to check the input.  When viewing, in case the
input is not a `List`, the proper result is `undefined`.  When updating the
proper way to handle a non-`List` is to treat it as empty and also to replace a
resulting empty list with `undefined`.  Also, when updating, we treat
`undefined` as a request to `delete` rather than `set`.

We can now view existing elements:

```js
const sampleList = Immutable.List(["a", "l", "i", "s", "t"])
L.get(idxList(2), sampleList)
// 'i'
```

Update existing elements:

```js
L.modify(idxList(1), R.toUpper, sampleList)
// List [ "a", "L", "i", "s", "t" ]
```

Remove existing elements:

```js
L.remove(idxList(0), sampleList)
// List [ "l", "i", "s", "t" ]
```

And removing the last element propagates removal:

```js
L.remove(["elems", idxList(0)], {elems: Immutable.List(["x"]), look: "No elems!"})
// { look: 'No elems!' }
```

We can also create lists from non-lists:

```js
L.set(idxList(0), "x", undefined)
// List [ "x" ]
```

And we can also append new elements:

```js
L.set(idxList(5), "!", sampleList)
// List [ "a", "l", "i", "s", "t", "!" ]
```

Consider what happens if the index given to `idxList` points further beyond the
last element.  The [`L.index`](#L-index) lens adds `null` values.  The above
lens adds `undefined` values, which is not ideal with partial lenses, because of
the special treatment of `undefined`.  In practise, however, it is not typical
to `set` elements except to append just after the last element.  Treating the
special case is left as an exercise for the reader.

#### Interfacing traversals

Fortunately we do not need Immutable.js data structures to provide a compatible
*partial*
[`traverse`](https://github.com/rpominov/static-land/blob/master/docs/spec.md#traversable) function
to support [traversals](#traversals), because it is also possible to implement
traversals simply by providing suitable isomorphisms between Immutable.js data
structures and JSON.  Here is a partial [isomorphism](#isomorphisms) between
`List` and arrays:

```js
const fromList = xs => Immutable.List.isList(xs) ? xs.toArray() : undefined
const toList = xs => R.is(Array, xs) && xs.length ? Immutable.List(xs) : undefined
const isoList = L.iso(fromList, toList)
```

So, now we can [compose](#L-compose) a traversal over `List` as:

```js
const seqList = [isoList, L.sequence]
```

And all the usual operations work as one would expect, for example:

```js
L.remove([seqList, L.when(c => c < "i")], sampleList)
// List [ 'l', 's', 't' ]
```

And:

```js
L.foldMapOf(Concat,
            [seqList, L.when(c => c <= "i")],
            R.toUpper,
            sampleList)
// 'AI'
```

## Background

### Motivation

Consider the following REPL session using Ramda:

```js
R.set(R.lensPath(["x", "y"]), 1, {})
// { x: { y: 1 } }
```
```js
R.set(R.compose(R.lensProp("x"), R.lensProp("y")), 1, {})
// TypeError: Cannot read property 'y' of undefined
```
```js
R.view(R.lensPath(["x", "y"]), {})
// undefined
```
```js
R.view(R.compose(R.lensProp("x"), R.lensProp("y")), {})
// TypeError: Cannot read property 'y' of undefined
```
```js
R.set(R.lensPath(["x", "y"]), undefined, {x: {y: 1}})
// { x: { y: undefined } }
```
```js
R.set(R.compose(R.lensProp("x"), R.lensProp("y")), undefined, {x: {y: 1}})
// { x: { y: undefined } }
```

One might assume that [`R.lensPath([p0,
...ps])`](http://ramdajs.com/docs/#lensPath) is equivalent to
`R.compose(R.lensProp(p0), ...ps.map(R.lensProp))`, but that is not the case.

In JavaScript, missing data can be mapped to `undefined`, which is what partial
lenses also do, because `undefined` is not a valid [JSON](http://json.org/)
value.  When a part of a data structure is missing, an attempt to view it
returns `undefined`.  When a part is missing, setting it to a defined value
inserts the new part.  Setting an existing part to `undefined` removes it.

With partial lenses you can robustly compose a path lens from prop lenses
`L.compose(L.prop(p0), ...ps.map(L.prop))` or just use the shorthand notation
`[p0, ...ps]`.

### Performance

Here are a few benchmarks on partial lenses (as `L` version 5.3.2) and some
roughly equivalent operations using [Ramda](http://ramdajs.com/) (as `R` version
0.22.1) and [Ramda Lens](https://github.com/ramda/ramda-lens) (as `P` version
0.1.1).

```jsx
L.foldMapOf(Sum, L.sequence, id, xs100) x  1,379,476 ops/sec ±0.50% (182 runs sampled)
P.sumOf(P.traversed, xs100)             x     23,977 ops/sec ±0.72% (181 runs sampled)
R.sum(xs100)                            x    143,341 ops/sec ±0.55% (180 runs sampled)

L.collect(L.sequence, xs100)            x    358,058 ops/sec ±0.49% (187 runs sampled)

L.modify(L.sequence, inc, xs100)        x  1,849,076 ops/sec ±0.54% (181 runs sampled)
P.over(P.traversed, inc, xs100)         x     14,117 ops/sec ±0.43% (182 runs sampled)
R.map(inc, xs100)                       x  1,930,424 ops/sec ±0.57% (177 runs sampled)

L.get(1, xs)                            x 35,445,315 ops/sec ±0.56% (180 runs sampled)
R.nth(1, xs)                            x  4,106,084 ops/sec ±0.45% (186 runs sampled)
R.view(l_1, xs)                         x  2,059,149 ops/sec ±0.57% (182 runs sampled)

L.set(1, 0, xs)                         x 22,716,967 ops/sec ±0.50% (179 runs sampled)
R.update(1, 0, xs)                      x  8,819,862 ops/sec ±0.50% (183 runs sampled)
R.set(l_1, 0, xs)                       x  1,416,881 ops/sec ±0.48% (181 runs sampled)

L.get("y", xyz)                         x 29,246,767 ops/sec ±0.73% (176 runs sampled)
R.prop("y", xyz)                        x 33,311,414 ops/sec ±0.56% (184 runs sampled)
R.view(l_y, xyz)                        x  4,295,269 ops/sec ±0.52% (188 runs sampled)

L.set("y", 0, xyz)                      x  7,436,057 ops/sec ±0.61% (179 runs sampled)
R.assoc("y", 0, xyz)                    x 13,977,977 ops/sec ±0.55% (184 runs sampled)
R.set(l_y, 0, xyz)                      x  2,048,229 ops/sec ±0.34% (181 runs sampled)

L.get([0,"x",0,"y"], axay)              x 12,715,889 ops/sec ±0.54% (180 runs sampled)
R.view(l_0_x_0_y, axay)                 x    683,161 ops/sec ±0.50% (183 runs sampled)

L.set([0,"x",0,"y"], 0, axay)           x  3,164,670 ops/sec ±0.64% (182 runs sampled)
R.set(l_0_x_0_y, 0, axay)               x    446,004 ops/sec ±0.53% (182 runs sampled)

L.modify([0,"x",0,"y"], inc, axay)      x  3,255,169 ops/sec ±0.65% (180 runs sampled)
R.over(l_0_x_0_y, inc, axay)            x    454,818 ops/sec ±0.43% (176 runs sampled)

L.remove(1, xs)                         x 23,454,934 ops/sec ±0.65% (179 runs sampled)
R.remove(1, 1, xs)                      x  7,790,301 ops/sec ±0.65% (175 runs sampled)

L.remove("y", xyz)                      x 13,355,385 ops/sec ±0.62% (179 runs sampled)
R.dissoc("y", xyz)                      x 14,044,862 ops/sec ±0.54% (180 runs sampled)

L.get(["x","y","z"], xyzn)              x 13,503,920 ops/sec ±0.58% (172 runs sampled)
R.path(["x","y","z"], xyzn)             x 15,959,699 ops/sec ±0.67% (174 runs sampled)
R.view(l_xyz, xyzn)                     x  3,696,731 ops/sec ±0.44% (180 runs sampled)
R.view(l_x_y_z, xyzn)                   x  1,409,591 ops/sec ±0.46% (179 runs sampled)

L.set(["x","y","z"], 0, xyzn)           x  2,887,232 ops/sec ±0.59% (180 runs sampled)
R.assocPath(["x","y","z"], 0, xyzn)     x  2,620,795 ops/sec ±0.58% (183 runs sampled)
R.set(l_xyz, 0, xyzn)                   x  1,234,385 ops/sec ±0.76% (182 runs sampled)
R.set(l_x_y_z, 0, xyzn)                 x    854,309 ops/sec ±0.53% (183 runs sampled)

L.remove(50, xs100)                     x  4,913,621 ops/sec ±0.47% (183 runs sampled)
R.remove(50, 1, xs100)                  x    998,646 ops/sec ±0.49% (182 runs sampled)

L.set(50, 2, xs100)                     x  4,959,245 ops/sec ±0.41% (184 runs sampled)
R.set(l_50, 2, xs100)                   x    801,953 ops/sec ±0.56% (182 runs sampled)
R.update(50, 2, xs100)                  x  1,686,352 ops/sec ±0.46% (177 runs sampled)
```

At the time of writing, various operations on *partial lenses have been
optimized for common cases*, but there is definitely a lot of room for
improvement.  The goal is to make partial lenses fast enough that performance
isn't the reason why you might not want to use them.

See [bench.js](./bench/bench.js) for details.

### Lenses all the way

As said in the first sentence of this document, lenses are convenient for
performing updates on individual elements of immutable data structures.  Having
abilities such
as [nesting](#L-compose), [adapting](#L-choose), [recursing](#L-lazy)
and [restructuring](#L-pick) using lenses makes the notion of an individual
element quite flexible and, even further, [traversals](#traversals) make it
possible to [selectively](#L-when) target zero or more elements
in [non-trivial](#L-branch) data structures a single operation.  It can be
tempting to try to do everything with lenses, but that will likely only lead to
misery.  It is important to understand that lenses are just one of many
functional abstractions for working with data structures and sometimes other
approaches can lead to simpler or easier
solutions.  [Zippers](https://github.com/polytypic/fastener), for example, are,
in some ways, less principled and can implement queries and transforms that are
outside the scope of lenses and traversals.

One type of use case which we've ran into multiple times and falls out of the
sweet spot of lenses is performing uniform transforms over data structures.  For
example, we've run into the following use cases:

* Eliminate all references to an object with a particular id.
* Transform all instances of certain objects over many paths.
* Filter out extra fields from objects of varying shapes and paths.

One approach to making such whole data structure spanning updates is to use a
simple bottom-up transform.  Here is a simple implementation for JSON based on
ideas from the [Uniplate](https://github.com/ndmitchell/uniplate) library:

``` js
const isObject = x => x && x.constructor === Object
const isArray = x => x && x.constructor === Array
const isAggregate = R.anyPass([isObject, isArray])

const descend = (w2w, w) => isAggregate(w) ? R.map(w2w, w) : w
const substUp = (h2h, w) => descend(h2h, descend(w => substUp(h2h, w), w))

const transform = (w2w, w) => w2w(substUp(w2w, w))
```

`transform(w2w, w)` basically just performs a single-pass bottom-up transform
using the given function `w2w` over the given data structure `w`.  Suppose we
are given the following data:

``` js
const sampleBloated = {
  just: "some",
  extra: "crap",
  that: [
    "we",
    {want: "to",
     filter: ["out"],
     including: {the: "following",
                 extra: true,
                 fields: 1}}]
}
```

We can now remove the `extra` `fields` like this:

``` js
transform(R.ifElse(isObject,
                   L.remove(L.props("extra", "fields")),
                   R.identity),
          sampleBloated)
// { just: 'some',
//   that: [ 'we', { want: 'to',
//                   filter: ['out'],
//                   including: {the: 'following'} } ] }
```

## Related work

Lenses are an old concept and there are dozens of academic papers on lenses and
dozens of lens libraries for various languages.  Here are just a few links:

* [Polymorphic Update with van Laarhoven Lenses](http://r6.ca/blog/20120623T104901Z.html)
* [A clear picture of lens laws](http://sebfisch.github.io/research/pub/Fischer+MPC15.pdf)
* [ramda/ramda-lens](https://github.com/ramda/ramda-lens)
* [ekmett/lens](https://github.com/ekmett/lens)
* [julien-truffaut/Monocle](https://github.com/julien-truffaut/Monocle)
* [xyncro/aether](https://github.com/xyncro/aether)

Feel free to suggest more links!
