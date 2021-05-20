# JS Promises and Async Programming

## The Callback Pyramid

Putting a callback function nested within a callback function is not ideal. It is referred to as a pyramid as subsequent nested functions form a (sideways) pyramid as shown below:

```javascript
a("test", (err, Result) => {
    b(aResult, (err, bResult) => {
        c(bResult, (err, cResult) => {
            d(cResult);
        }
    }
})
```

While the code above looks straightforward, it can easily become unreadable and difficult to debug. So how do we handle promises at the moment we need them?

First, let's explore what a promise is...

## Promise

A **promise** is an object that represents the eventual completion (or failure) of an asynchronous operation, and its resulting value.

Promises were introduced to make asynchronous code more readable.

Generally speaking, promises have three states:

1. Pending
2. Fulfilled
3. Rejected

(2) and (3) above may collectively also be referred to as *settled* or *resolved*.

### Axios

**Axios** is a popular http request library. It is an abstraction on top of `XMLHttpRequest`. Axios makes use of promises.

## Consuming Promises

Once a promises succeeds, it will call `then()` which itself takes a function as a parameter. Typically, we will put an anonymous function that will make of the returned data in some way.  Here is an example use:

```javascript
export function get() {
    axios.get("http://localhost:3000/orders/1")
    .then(({data}) => {
        setText(JSON.stringify(data));
    });
}
```

Note that the callback function in `then()` destructures the returned data payload as `{data}`. When consuming web APIs, there are often many things, such as headers, contained within the returned data. We only want the payload, which is typically labeled as `data`. Next, we stringify and pass that data into the `setText()` function.

### Handling Errors

Errors do **not** call the `then()` function. They will call the `catch()` function, which, like the `then()` function, can be chained after an axios `get()` request. Whatever the reason, the `catch()` function will pass into its callback the reason for the error which by convention we call "err".

### Chaining Promises

Promises return promises. That is, both `then()` and `catch()` return promises. This allows us to chain promises. See below for an example:

```javascript
export function chain() {
    axios.get("http://localhost:3000/orders/1").then(({ data }) => {
        return axios.get(`http://localhost:3000/addresses/${data.shippingAddress}`);
    })
    .then(({ data }) => {
        setText(`City: ${data.city}`);
    });
}
```

Note that it is very important that `then()` is called on an object. That is, it is easy to forget to add `return` in the first `then()` call. Had we forgotten, JS would return `undefined` (JS default object when it is expecting a return object but none is explicitly returned).

To handle errors, simply add the `catch()` function at the end of the chain. `catch()` will signal any errors it encounters during the chain.

## Creating Promises

What if wanted to create our own promises?

### Deeper Dive into Promise States

If the associated promise has already been resolved, either to a value, a rejection, or another promise, this method [resolve] does nothing. I.e., a settled promise is *done*.

The `Promise` object takes two important parameters: resolve and reject.

### Power of Asynchronous Programming

Promises allow us to asynchronously make simultaneous calls and create a callback that will be called once all those promised calls are returned.

To do this with promises, we can use the `all()` method. A drawback, however, is that the `all()` method will error out if even one promise returns with an error. What if we want to return whatever promises came back resolved and deal with the errors on the side? We use `allSettled()`.

### Race

Another advantage of promises is taking advantage of race conditions. Here, we can invoke `race()` to call on two or more promises and resolve on whichever returns first. That way, we can create multiple instances of an API and simply resolve on the one that returns the quickest.

## Async/Await

A few years after Promises were introduced, Async/Await was devised. Async/Await is syntactic sugar on top of promises which itself was a way to deal with asyronchronous programming more easily.

Recall: syntactic sugar within programming is designed to make things easier to read or to express.

### Error Handling in Async/Await

No new syntax is introduced, instead we use the familiar try/catch block.

### Chaining Async/Await

Just like in promises, we can chain async/await. Note that in chaining, we are making sequential calls (that is, order matters!).

### Awaiting Concurrent Requests

If we don't care for order and want to fire off any number of asynchronous calls, we can do that with async/await just like we did with promises. To do this, make any number of calls but omit `async` from them. For example:

```javascript
export async function concurrent(){
    const orderStatus = axios.get("http://localhost:3000/orderStatuses");
    const orders = axios.get("http://localhost:3000/orders");

    setText("");

    const {data: statuses} = await orderStatus;
    const {data: order} = await orders;

    appendText(JSON.stringify(statuses));
    appendText(JSON.stringify(order[0]));
}
```

Axios will go ahead and start to retrieve both `orderStatus` and `orders`. The next two lines after `setText()` show that we wait for `orderStatus` first.

### Awaiting Parallel Calls

## Questions

Not quite clear how to do error handling with chained promises. Will the last catch block display any and all errors of the chain?

Chaining vs. Concurrent vs. Parallel Async/Await?