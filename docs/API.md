# Aix API Reference

## Classes

- [DeferredIterable](#deferrediterable)
    - [DeferredIterable.callback](#deferrediterablecallback)
    - [DeferredIterable.iterable](#deferrediterableiterable)
    - [DeferredIterable.finally](#deferrediterablefinally)
    - [DeferredIterable.value](#deferrediterablevalue)
    - [DeferredIterable.finish](#deferrediterablefinish)

## Functions

- [of](#of)
- [map](#map)
- [filter](#filter)
- [flatMap](#flatMap)
- [concat](#concat)
- [reduce](#reduce)
- [merge](#merge)
- [insert](#insert)
- [tap](#tap)
- [zip](#zip)

# Classes

## DeferredIterable

`DeferredIterable` makes it easy to turn stream of events into an interable.

You typically interact with `DeferredIterable` in 2 ways:

- To send data into the `DeferredIterable` call the `callback` function to send an `IteratorResult`
- To read data from the `DeferredIterable` use the `iterable` property.

### Example

```javascript
import { DeferredIterable } from "aix/deferredIterable";

const deferredIterable = new DeferredIterable();

// set up a callback that calls value on the deferredIterable
const callback = value => deferredIterable.value(value);

// attach the callback to the click event
document.addEventListener('click', callback);

// remove the callback when / if the iterable stops
deferredIterable.finally(() => document.removeEventListener('click', deferredIterable.value));

// go through all the click events
for await (const click of deferredIterable.iterable) {
    console.log('a button was clicked');
}
```

### DeferredIterable.callback

The callback to call to send a `IteratorResult` into the `DeferredIterable`. An `IteratorResult`
has a `done` boolean property and a `value` property.

```javascript
import { DeferredIterable } from "aix/deferredIterable";

const deferredIterable = new DeferredIterable();
deferredIterable.callback({ done: false, value: 1 });
deferredIterable.callback({ done: false, value: 2 });
deferredIterable.callback({ done: false, value: 3 });
deferredIterable.callback({ done: true });

for await (const item of deferredIterable.iterable) {
    console.log(item); // prints 1, 2, 3
}
```

### DeferredIterable.iterable

An ```AsyncIterable``` that returns values supplied by calling ```callback```.

```javascript
import { DeferredIterable } from "aix/deferredIterable";

const deferredIterable = new DeferredIterable();
deferredIterable.callback({ done: false, value: 1 });
deferredIterable.callback({ done: false, value: 2 });
deferredIterable.callback({ done: false, value: 3 });
deferredIterable.callback({ done: true });

for await (const item of deferredIterable.iterable) {
    console.log(item); // prints 1, 2, 3
}
```

### DeferredIterable.finally

The callback supplied to ```finally``` is called when the iterable finishes either 
because it's run out of values, it has returned or an error was thrown.

### DeferredIterable.value

A helper method to pass a value to the DeferredIterable. Calling
```deferredIterable.value('test')``` is the same as calling 
```deferredIterable.callback({ done: false, value: 'test'} )```.

### DeferredIterable.finish

A helper method to signal the last value to DeferredIterable. Calling
```deferredIterable.finish()``` is the same as calling 
```deferredIterable.callback({ done: true} )```.

# Functions

## of

Construct a new async iterable from a series
of values.

```javascript
import { of } from "aix/of";

const values = of(1, 2, 3);

for await(const item of mapped) {
    console.log(item); // outputs 1, 2, 3
}
```

## map

Go through each item in the iterable and run a mapping function. The result will be a new
iterable with the transformed values.

```javascript
import { map } from "aix/map";
import { of } from "aix/of";

const mapped = map(value => value * 2)(of(1, 2, 3));

for await(const item of mapped) {
    console.log(item); // outputs 2, 4, 6
}
```

## flatMap

Go through each item in the iterable and run a mapping function that returns an async iterable. The
result is then flattened.

```javascript
import { flatMap } from "aix/flatMap";
import { of } from "aix/of";

const mapped = flatMap(
    async function* (value) {
        yield value;
        yield value;
    }
)(of(1, 2, 3);

for await(const item of mapped) {
    console.log(item); // outputs 1, 1, 2, 2, 3, 3
}
```

## filter

Filter an iterable based on some criteria.

```javascript
import { filter } from "aix/filter";
import { of } from "aix/of";

const filtered = filter(
    value => value % 2 === 0
)(of(1, 2, 3, 4, 5, 6);

for await(const item of filtered) {
    console.log(item); // outputs 2, 4, 6
}
```

## concat

Concatenate 2 iterables in order

```javascript
import { concat } from "aix/concat";
import { of } from "aix/of";

const concatted = concat(
    of(1, 2)
)(of(3, 4);

for await(const item of concatted) {
    console.log(item); // outputs 1, 2, 3, 4
}
```

## reduce

Reduce a series of values to a single result. The series of values is
reduced by a function that compbines a running total or accumulator with
the next value to produce the new total or accumulator.

```javascript
import { reduce } from "aix/reduce";
import { of } from "aix/of";

const reduced = concat(
    (accumulator, next) => accumulator + next, // sum the values together
    0
)(of(1, 2, 3));

console.log(reduced); // 6
```

## merge

Merge a number of async iterators into one concurrently. Order is not important.

```javascript
import { merge } from "aix/merge";
import { of } from "aix/of";

const merged = merge(
    of(1, 2), of(3, 4)
);

for await(const item of concatted) {
    console.log(item); // outputs 1, 2, 3, 4 in no particular order
}
```

## insert

Insert values at the beginning of an async iterable.

```javascript
import { insert } from "aix/insert";
import { of } from "aix/of";

const inserted = insert(
    1, 2, 3
)(of(4, 5, 6));

for await(const item of concatted) {
    console.log(item); // outputs 1, 2, 3, 4, 5, 6
}
```

## tap

'Taps' an async iterable. Allows you to run a function for
every item in the iterable but doesn't do anything with the 
result of the function. Typically used for side effects like
logging.

```javascript
import { tap } from "aix/tap";
import { of } from "aix/of";

const tapped = tap(
    value => console.log(value) // prints 1, 2, 3
)(of(1, 2, 3));

for await(const item of concatted) {
    console.log(item); // prints 1, 2, 3
}
```

## zip

Creates a new iterable out of the two supplied by pairing up equally-positioned items from both iterables. The returned iterable is truncated to the length of the shorter of the two input iterables.

```javascript
import { zip } from "aix/zip";
import { of } from "aix/of";

const zipped = zip(
    of(1, 2)
)(of(1, 2));

for await(const item of concatted) {
    console.log(item); // prints [1, 1], [2, 2]
}
```
