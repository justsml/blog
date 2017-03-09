# Idea Book 

> Work In Progress

```js
// For situations where you need to promisify over odd shaped API's (streams, multiple fail events conditions)
// this is usually an antipattern, but may come in handy:
function getFlatPromise() { 
  var resolve, reject, promise = new Promise((_resolve, _reject) => {
    resolve = _resolve;
    reject  = _reject;
  })
  return [promise, resolve, reject];
}


```
