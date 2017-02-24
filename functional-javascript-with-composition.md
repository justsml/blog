# Functional JavaScript Patterns



One of the most important ideas in software development is: keep it simple.

The problem is, every developer believes all they write is simple (& beautiful) code.
... I'm going to avoid wading into the tarpit of defining "Simple Code."

Instead I'll show 2 'rules' which have helped me write more testable & adaptable code. (Which I feel are key to Simple code)

1. Restrict functions to single-purpose. (Single Responsibility Principle)
2. Restrict functions to single-argument (or 2 for (err, value) style).
This can be an array or object with many-faceted data.

You may be wondering how single-purpose functions ever amount to anything useful, well, let me introduce you to my friend, Higher Order Components, or HOCs. HOC is really a fancy way of saying Function. 
You may know HOCs by other names: The elaborate excuse for what I used to call my "Controllers," are now discreet wrappers of function chains.

Let's do a few functions with simple math.
Using Pure ES2016 - and a tape test.

Includes examples w/ traditional functions, built-ins (Array.reduce), and promises.



```js
/*
@author: Dan Levy <Dan@DanLevy.net>
Twittering: @justsml
*/
const test          = require('tape');

// The '_composer' wrapper fn supports reversing execution order, literally only changes the inner fn '_partial'
const compose       = _composer();      // note: '_composer' defined at the end, read tests first
const composeRight  = _composer(true);

// Define a Pure-at-heart function (https://en.wikipedia.org/wiki/Pure_function)
const add5 = n => {
  n = parseInt(n) + 5  
  return n; // required
}
test('sequence of 3 functions', t => {
  const add15 = compose(add5, add5, add5);
  t.equals(add15(0), 15)
  t.equals(add15(5), 20)
  t.equals(add15('5'), 20)
  t.end();
})

// Define some Pure-at-heart functions (https://en.wikipedia.org/wiki/Pure_function)
const half = n => {
  n = (parseInt(n) * 0.5).toFixed(2)
  return n; // required
}
const square = n => {
  n = (parseInt(n * n)).toFixed(2)
  return n; // required
}

/*
Code reuse & independence across varying patterns    #AllTheMonads
*/
test('composition: math functions', t => {
  const add5HalfSquare = compose(add5, half, square);
  t.equals(add5HalfSquare(5), '25.00', 'I can caz maths?');
  //       5+5==10,  half(10)==5,  5*5==25.0
  t.end();
})
test('promises: math functions', t => {
  const add5HalfSquare = n => Promise.resolve(n)
    .then(add5).then(half).then(square);

  add5HalfSquare(5)
  .then(answer => {
    t.equals(answer, '25.00', 'I can promise maths?');
    //       5+5==10,  half(10)==5,  5*5==25.0
    t.end();
  });
})
test('array.reduce: math functions', t => {
  const add5HalfSquare = n => [add5, half, square]
    .reduce((val, fn) => fn && fn(val), n);
  t.equals(add5HalfSquare(5), '25.00', 'I can reduce maths?');
  //       5+5==10,  half(10)==5,  5*5==25.0
  t.end();
})




// Plz ignore any 'fanciness' here - the point is the pattern above:
// Forward/reverse function chain helper - tldr: this glues other functions toghether
function _composer() {
  const fns = Array.from(arguments);
  return function _partial() {
    return Array.from(fns).reduce((last, fn) => fn && fn(last) || last, [...arguments]);
  }.bind(this);
}
```


You may be thinking it's easy to write like that when it's just dummy functions. C'mon `add5`!



So, I'm going to go through a login function for a chat app.

Mock Requirements:
1. User clicks login.
2. Modal prompts for `user` and `pass` fields.
    2a. User submits form.
    2b. User clicks 'forgot password'.
3. Upon success, load app+msg data for logged-in user.
    3a. Cache ajax data for offline use.
    3b. Update user status to 'online'
4. Upon failure, plunge into fire pit.
5. Show appropriate UI messaging

Using bluebird's unique take on "Promises," I'll show the code in *almost* as many lines as the (terse) requirements above.

chatApp.login = () => openLoginModal()
    .then({user, pass}) => ajaxLogin({user, pass}))
    .then(user => {
        setUserStatus('online') // < non blocking, as result is irrelevant.
        return [getContacts(user), getRooms(user), getMessages(user)]
    })
    .then(([contacts, rooms, messages]) => {
        alertOnNew({contacts, messages})
        return this.cache({contacts, rooms, messages})
    })
    .catch(UserCancel, hideModal)
    .catch(ForgotPassword, () => showUserMessage({message: 'think harder'}))
    .catch(LoginError, compose(hideModal, initFirePit, destroyApp))
    .catch(err => showUserMessage({message: 'Something truly unexpected happened, congratulations.'}))

========
^^^ Code modified from a real-life app. Sadly the 'fire pit' feature is still in backlog.

I try organize succinct 'pathways' in my code. 
Your Intent & flow must be fairly obvious. If your logic is 3 levels deep, nevermind across 3 files, you've lost 90% of "developers." 
Good APIs are easily understood & implemented. 
The Best APIs are built atop a stack of good APIs. 

========  CONTINUED ========


You may have noticed something else about my code - 2 guiding principles (if not strict rules) for which I aim.

1. Eschew nested logic - a sign you are mixing 2+ different things. It's also a sign of untestable/high dimensionality.
2. More than 2 lines per function? Stop and think, or ask your nearest dev. Or you must be writing OpenSSL.

Ok, I'm being a bit of a troll about the '2 line limit.' 

Here is what I'm getting at - we can gain more testable code if we refactor like so:

```js
chatApp.getUserData = user => {
    return [getContacts(user), getRooms(user), getMessages(user)]
}
chatApp.login = () => openLoginModal()
    .then({user, pass}) => ajaxLogin({user, pass}))
    .tap(() => setUserStatus('online'))
    .then(chatApp.getUserData)
    .tap(([contacts, rooms, messages]) => alertOnNew({contacts, messages}))
    .then(this.cache)
    .catch(UserCancel, hideModal)
    .catch(ForgotPassword, () => showUserMessage({message: 'think harder'}))
    .catch(LoginError, compose(hideModal, initFirePit, destroyApp))
    .catch(err => showUserMessage({message: 'Something truly unexpected happened, congratulations.'}))
```

It's better. Here's why:
`chatApp.getUserData` is now a testable function, instead of hidden inside the login() & coupled to the status update.
It's also flat.
Partitioned into 2 'sections' - .then/.tap, and then .catch's. 
Errors can be filtered by type in bluebirds' .catch(<type>, <error>).
This creates a clear declaration of how your code ought to behave. 
Either a .catch() function fired, or you get the result RETURNED from the final .then(). 



> The more complex logic your app belongs in the farthest depths of your code tree. Break your tasks apart until they can resemble readable sequential chains.

You will find what "code reuse" *really* means. (In school I learned it's reusing the same bad pattern)
The clouds will clear, sun will shine, rainbows and ... you get the idea.

