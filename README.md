# work-queue
A light-weight in-memory NodeJs queue for throttling asynchronous work items that implement [Q-like](https://www.npmjs.com/package/q) promise

## Usage
```js
var WorkQueue = require('work-queue');

// Create the queue
var requestQueue = new WorkQueue({
  concurrency: 2,
  callback: function(method, url) {
    return http.request(method, url) // Returns a Q-like response
      .then(function(resp) {
        console.log('response = ', resp.text);
      });
  }
});

// Add some work items
for (var i = 0; i < 20; i += 1) {
  requestQueue.push('GET', 'https://github.com/MikeyBurkman/work-queue');
}

// Keep checking, and exit when the queue is empty
setInterval(function() {
  if (requestQueue.empty()) {
    console.log('Complete!');
    process.exit(0);
  }
}, 1000);
```

## Description of the Example
- A maximum of 2 concurrent requests (`concurrency: 2`) will be happening at any given time. 
- When the queue can process work, the `callback` function will be called.
- The `push(args)` method on the queue puts work items on the queue. 
- The `empty()` returns true if the queue has finished processing all the work given to it.

## Constructor Options
`concurrency`: The maximum number of items that can be processed concurrently.

`callback`: The function to be executed when a work item can be processed.
- The callback function is expected to return a [Q-like](https://www.npmjs.com/package/q) promise in order to throttle correctly.
- If the callback function returns something else, it is assumed to be finished as soon as the function returns.

`interval`: When there are items on the queue, the queue will check every `interval` milliseconds to see if it can start processing a new work item.

## API
`push(args)`: Pushes a work item on to the queue.
- Any arguments may be given. The arguments given to `push` are given to the `callback` function exactly.

`empty()`: Returns true if the queue is empty and is not currently processing anything.

`notEmpty()`: The inverse of `empty()`.

`clear()`: Clears the queue. 
- Anything currently processing will finish, but no more work items in the queue will be processed.
