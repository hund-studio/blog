---
title: Array iteration examples
description: 'How to handle data using map, every, find, filter, reduce, sort, every with real examples'
tags: ''
cover_image: ''
canonical_url: null
published: false
id: 1264754
---

This article tries to examine different real-life use cases which can be solved with `Array` methods.
For a technical definition of each method, every section has a link to the official **📄 MDN Reference**. This article is about examples and tips, not documentation.

|       | Table of contents                                 |
| :---: | :------------------------------------------------ |
|       | [`map()` ](#map)                                  |
|       | [`every()`](#every)                               |
|       | [`find()`](#find)                                 |
|       | [`filter()`](#filter)                             |
|       | [`reduce()`](#reduce)                             |
|       | [`sort()`](#sort)                                 |
|       | [`every()`](#every)                               |
| BONUS | [The spread iterator `...`](#the-spread-operator) |

## Map

> 📄 [MDN REference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)

`map()` is probably the most used and versatile method out there, as MDN states:

> Returns a new array containing the results of invoking a function on every element in the calling array.

### Execute the same function on every item of the array

> 👨‍🏫 You have a bunch of data and wants to run the same action on all of them.

```typescript
const numbers = [1, 2, 3];

const numbersPlusOne = numbers.map((number) => number + 1);

// expected:
// numbersPlusOne: 2, 3, 4
```

### Convert an array of configuration options to Class Instances

> 👨‍🏫 You have an array with different arguments and want to change them into Class Instances.

```typescript
const peopleOptions = [
  {
    name: "Antonio",
    age: 13,
    height: 1.83,
  },
  {
    name: "Federico",
    age: 54,
    height: 0.9,
  },
  {
    name: "Ernesto",
    age: 27,
    height: 2.2,
  },
];

const people = peopleOptions.map((personOptions) => new Person(personOptions));

// expected:
// people: 👦, 👩, 👱
```

### In React, render a list of wathever object

> 👨‍🏫 You want to render some UI elements using React.

```typescript
// ... some React Component logic

const list = ["milk", "eggs", "cucumbers"];

return (
  <ul>
    {list.map((item, index) => (
      <li key={index}>{item}</li>
    ))}
  </ul>
);
```

### Bonus Track: In combination with Promise.all()

In combitaion with `Promise.all()` can be used to wait for multiple `async` operations BUT:

> ⚠️ When using `Promise.all()` the asynchronous functions are not executed in the same order of the initial array. If you have asynchronous functions which need to run in an **exact** sequence (ae. update a record after a creation) you must use `forEach()` which will wait for each loop to be complete before continue.

> ⚠️ Even though the asynchronous functions are executed in a random order the output results array will preserve the same order of the initial array.

```typescript
const membersName = ["Antonio", "Ernesto", "Federico"];

const membersAge = await Promise.all(
  members.map(async (memberName) => {
    const age = await getAgeFromDbWhereNameIs(memberName);

    return age;
  })
);

// Wait untill Promise is fullfilled

// expected:
// membersAge: 13, 27, 54
```

## Every

> 📄 [MDN Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/every)

This method can be used to check wether all elements inside an array fullfill a specific requirement.
This method is the only one which does not return an array whereas a boolean.

> 👨‍🏫 You want to filter an array and detect which items for a specific order are in stock and ready for shipment.

```typescript
const order = [
  {
    name: "Tea Shirt",
    availableStock: 10,
  },
  {
    name: "Jeans",
    availableStock: 0,
  },
  {
    name: "Filp Flops",
    availableStock: 3,
  },
];

const isOrderReadyForShipment = order.every(
  (product) => product.availableStock > 0
);

// expected:
// isOrderReadyForShipment: false
```

## Find

> 📄 [MDN Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find)

This method can be used to find the **first** element which match a condition.

> ⚠️ This method will only return THE FIRST matching element, if there is more than one element which match the condition it will be ignored.

> 👨‍🏫 You have a list of posts and want to get the **first** matching the tag **yes**.

```typescript
const posts = [
  {
    title: "How to open your mind using a knife",
    author: "Ernesto",
    tag: "yes",
  },
  {
    title: "Let your dog do wathever he wants",
    author: "Federico",
    tag: "no",
  },
  {
    title: "How to escape from a glass cage",
    author: "Ernesto",
    tag: "yes",
  },
];

const yesPost = posts.find((post) => post.tag === "yes");

/**
 * expected:
 * yesPost:  {
 *    title: "How to open your mind using a knife",
 *    author: "Ernesto",
 *    tag: "yes",
 *  },
 */
```

## Filter

> 📄 [MDN Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)

Now that you are a `find` expert we can bring it to the nex level. The `filter` method works the same as `find` but it returns all matching values instead.

It can be used for a couple of things such as...

### Filter values that match certain criteria

> 👨‍🏫 You have a list of posts and want to get all posts matching the tag **yes**.

```typescript
const posts = [
  {
    title: "How to open your mind using a knife",
    author: "Ernesto",
    tag: "yes",
  },
  {
    title: "Let your dog do wathever he wants",
    author: "Federico",
    tag: "no",
  },
  {
    title: "How to escape from a glass cage",
    author: "Ernesto",
    tag: "yes",
  },
];

const yesPosts = posts.filter((post) => post.tag === "yes");

/**
 * expected:
 * yesPost: [
 *  {
 *     title: "How to open your mind using a knife",
 *     author: "Ernesto",
 *     tag: "yes",
 *   },
 *  {
 *     title: "How to open your mind using a knife",
 *     author: "Ernesto",
 *     tag: "yes",
 *  }
 * ]
 */
```

### Clean an array from unwanted/falsy values

```typescript
const array = [null, undefined, false, "hello", null, -1, "world"];

const arrayWithoutFalsyValues = array.filter((item) => !!item);

// expected:
// arrayWithoutFalsyValues: hello, world
```

When wokring with React you might have notced that when using classes to style `is-active` state sometimes you get dirty class names:

- `" undefined"`;
- `"false null another-class-name"`.

> 👨‍🏫 You want to create a function to clean classNames from unwanted/falsy values.

```typescript
const active = false;

return <div className={[active && "is-active", "round-borders"].join(" ")} />;
// expected <div class="false round-borders">

const css = (...classNames: (string | undefined | null | false)[]) =>
  classNames.filter((className) => !className).join(" ");

return <div className={css(active && "is-active", "round-borders")} />;
// expected <div class="round-borders">
```

## Sort

> 📄 [MDN Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)

With the `sort` method things are getting complicated.
You have an array with 3 numbers `[3, 18, 7, 18]` which needs to be sorted in ASCending order.

Sort method compare values two by two and depending on the value obtained subtracting one from another decide wether:

- (3 - 18) = -15 which is `< 0` which means keep 3 before 18 => new sequence `[3, 18, 7, 18]`;
- (18 - 7) = +11 which is `> 0` which means exchange 18 with 7 => new sequence `[3, 7, 18, 18]`;
- (18 - 18) = 0 which is `= 0` which means keep the same order => new sequence `[3, 7, 18, 18]`.

```typescript
const numbers = [3, 18, 7];

const sortedNumbers = numbers.sort((current, next) => curremt - next);

// expected:
// sortedNumbers: 3, 7, 18
```

The same method works with any kind of data:

### Sorting dates

```typescript
const events = [
  { name: "Social pic-nic", date: "12/11/2022" },
  { name: "Karate Lesson", date: "3/8/2022" },
  { name: "Pay Rent", date: "23/12/2022" },
];

const sortedEvents = events.sort((current, next) => current.date > next.date);

// expected:
// sortedNumbers: Karate Lesson, Social pic-nic, Pay Rent
```

### Sorting alphabetical order

```typescript
const words = ["alpha", "beta", "gamma"];

const sortedWords = events.sort((current, next) => current > next);

// expected:
// sortedWords: alpha, bbeta, gamma
```

## Reduce

> 📄 [MDN Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)

Reduce is so far the most complex but the most versatile method.
That said it can be used to run a function trough every element of the array.

### Get the sum of a specific item key value inside an array

> 👨‍🏫 Calculate the subtotal of all items inside a cart.

```typescript
const cart = [
  {
    name: "T-shirt",
    qty: 1,
    price: 10000,
  },
  {
    name: "Hat",
    qty: 2,
    price: 20000,
  },
  {
    name: "T-shirt",
    qty: 1,
    price: 15000,
  },
];

const cartSubtotal = cart.reduce((cartSubtotal, item) => {
  const itemSubtotal = item.qty * item.price;
  return cartSubtotal + itemSubbtotal;
}, 0);
```

### Manipulate data inside objects

In combination with `Object.keys()` the `reduce` method can be used to loop trough javascript obbjects and manipulate their data.

> 👨‍🏫 Create two functions to remove or add given quantities of items from a cart.

> ⚠️ This `cart` variable is different from the previous. Previous one was an array, this is an object.

```typescript
const cart = {
  productCodeSample1234: {
    name: "T-shirt",
    qty: 1,
    price: 10000,
  },
  sampleuniqueCode1234: {
    name: "Hat",
    qty: 2,
    price: 20000,
  },
  otherCodeSample01234: {
    name: "T-shirt",
    qty: 1,
    price: 15000,
  },
};

const addToCart = (uniqueCode: string, qty: number | undefined = 1) =>
  Object.keys(cart).reduce((cart, cartProductCode) => {
    if (cartProductCode === uniqueCode) cart[uniqueCode].qty += qty;
    return cart;
  }, cart); // add a given qty of products from your cart

const removeFromCart = (uniqueCode: string, qty: number | undefined = 1) =>{
  const updateItemsQuantity = Object.keys(cart).reduce((cart, cartProductCode) => {
    if (cartProductCode === uniqueCode) cart[uniqueCode].qty -= qty;
    return cart;
  }, cart); // remove a given qty of products from your cart}

  const updateItemsQuantity.filter(item => item.qty > 0); // remove ites with 0 or negative qty
```

## The spread operator

Even tough spread operator it is not directly realated to array methods, rather than to all generic iterable objects, I thought it would have been useful to mention it in this compendium.

### Add elements to an array

The `...` spread operator basically means: repeat the items of an array (or the object keypairs) inside a new iterable object.
This allow us to clone an array inside another one while we add new items in between.

```typescript
const lunchbbox = ["apple", "banana"];
const pencilCase = ["pen", "pencil"];

const backpack = ["tablet", ...lunchBox, "laptop", ...pencilCase];

// expected:
// backpack = ['tablet', 'apple', 'banana', 'laptop', 'pen', 'pencil']
```

> 👨‍🏫 If we need to create one function to **add** items from an array we will use the `...` spread operator whereas we will use a `filter` method to **remove** them.

### Override a default confguration

When you create a slider you probably set some default options (such as animation speed, transition type ...) which can be override from annother developer in order to costumize it.

```typescript
interface SliderConfig {
  animationSpeed: number;
  transitionType: "fade" | "slide",;
}

const defaultConfig: SliderConfig = {
  animationSpeed: 0.2,
  transitionType: 'slide'
};

const applyUserConfig = (userConfig: SliderConfig) => {
  return {
    ...defaultConfig,
    ...userConfig,
  };
};

const sliderConfig = applyUserConfig({
  animationSpeed: 0.4
})

/**
 * expected:
 * {
 *    animationSpeed: 0.4
 *    transitionType: 'slide'
 * }
```

### Repeat arguments of concatenadted complex functions

This is a complex and probably uncommon case but it can be usefull to store it somewhere in your mind when difficult times will come.

Spread operator can be used to spread function arguments with a function which accepts a subset of given args. Its only need is to make the code more readable BUT it will require more attention because it reduce how freely you can edit and change the structure of your functions...as I said _store it somewhere in your mind when difficult times will come_.

```typescript
const createAChocolateEgg = (
  height: number,
  width: number,
  chocolateType: string
) => {
  return `A ${chocolateType} chocolate egg, height: ${height}, width: ${width}`;
};

const createADecoratedChocolateEgg = (
  color: string,
  ...chocolateEggArgs: [number, number, string]
) => {
  return `${createAChocolateEgg(
    ...chocolateEggArgs
  )} whith ${color} sprinkles on it.`;
};
```
