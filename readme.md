<!--
Market: SF
-->

![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png)

<!-- 9:00 5 minutes -->

<!--Hook: So by now we've hammered in the concept of the request response cycle pretty hard.  Think for a moment about how you would summarize it to a new web developer.  Now turn to your partner and share your summary.  Who wants to share their summary?  [Once summary is established, move into lecture] -->

# Angular $http

### Why is this important?
*This workshop is important because:*

As you have been learning, Angular has its own way of doing things. $http is how Angular handles web requests. It is to Angular what $.ajax, $.get, $.post etc. is to jQuery..  

### What are the objectives?
*After this workshop, developers will be able to:*

- **Use** $http to access an API resource, rather than use hardcoded data.

### Where should we be now?
*Before this workshop, developers should already be able to:*

- **Start up** a Node.js app
- **Create** an Angular app with controllers
- **Understand** AJAX & RESTful routing

<!-- 9:05 5 minutes -->

## Intro

We've only been working with hardcoded data so far. Today that changes; it's time to kick it up a notch.

![](http://piqueyeater.files.wordpress.com/2013/12/emeril-bam-gif.gif)

We're going to learn a little about two different functionalities in Angular that will allow us to start communicating with real data, accessed through an API. You'll need to dust off your knowledge of RESTful routes & AJAX, but hopefully that's a good thing.

We are going to be using an external JSON API. Next week we will go over making strictly JSON API's in both Rails and Express, but today we will use a  little Node API we built for you.

Now, real quick – we might want a little seed data. Take a minute and make some POST requests in CURL, Postman, etc. to add some presidents to our database. 

<details>
<summary>If you need some examples:</summary>

```json
[
  {"name": "George Washington", "start": 1789, "end": 1797 },
  {"name": "John Adams", "start": 1797, "end": 1801 },
  {"name": "Thomas Jefferson", "start": 1801, "end": 1809 },
  {"name": "James Madison", "start": 1809, "end": 1817 }
]
```
</details>

<details>
<summary>curl example:</summary>
```bash
curl -H "Content-Type: application/json" -X POST -d '{"name": "George Washington", "start": 1789, "end": 1797 }' http://localhost:3000/presidents
{"president":{"__v":0,"name":"George Washington","start":1789,"end":1797,"_id":"56f18463508799dc44f2fbee"}}
```
</details>

Once you have some, do a quick `GET` request to `http://localhost:3000/presidents` and make sure you've got some JSON.

<!-- 9:10 < 5 minutes -->

## Demo of Starter Code

Okay, so we've included a bunch of starter code that looks quite a bit like the code you've already written. There's a controller, with some hardcoded data, listing out some of the Presidents in the United States.

It's our job to connect this little API we have, and our Angular application.

<img width="752"  src="https://cloud.githubusercontent.com/assets/25366/9017871/7cf4a79e-378e-11e5-85d8-d018f0a7ab21.png">

<!--Do this at half-mast then turn over to devs-->

<!-- 9:10 35 minutes -->

## Hitting an API with $http

The simplest starting point will be to switch our hardcoded array of presidents with the one living in our new API.

Step one – **let's delete our hardcoded data.** In `presidentsController.js`:

```diff
angular.module('ThePresidentsApp', [])
  .controller('PresidentsController', PresidentsController);

function PresidentsController(){
-  this.all = [
-    {name: 'George Washington', start: 1789, end: 1797 },
-    {name: 'John Adams', start: 1797, end: 1801 },
-    {name: 'Thomas Jefferson', start: 1801, end: 1809 },
-    {name: 'James Madison', start: 1809, end: 1817 }
-  ]
+  this.all = [];
}
```

With a little setup, we'll do a GET request to our API, and assign `this.all` to the array we get back. To do that, we're going to have to use an Angular library called `$http`.

### Injecting Dependencies

Angular dependencies – like libraries or plugins that other people have built – are defined first in our module (unless they come with Angular by default), and then _injected_ into any controllers that need to use them.

`$http` happens to come with Angular, so we only need to _inject_ it into our controller. We do that with a simple command, and then by simply passing an argument to our controller function.

In `js/presidentsController.js`:
```js
PresidentsController.$inject = ['$http'];
function PresidentsController($http){
  // ...
```

The first tells the controller we intend to use this library called `$http`, the second allows us to pass the library in and gives it the name $http. Think of it just like any other argument in a function – because it's the first argument, and we called it $http, we can use it inside our function using that name.

### Using $http is just AJAX!

`$http` is not very different than how we've used AJAX in the past, especially with JQuery. Let's see it all, then walk through it. In `js/presidentsController.js` again:

```js
PresidentsController.$inject = ['$http'];

function PresidentsController($http){
  var self = this;
  self.all = [];

  function getPresidents(){
    $http
      .get('http://localhost:3000/presidents')
      .then(function(response){
        self.all = response.data.presidents;
    });
  }

  getPresidents();

// ...
}
```

There are a few important things to note. Let's cut it down first just to $http:

```js
function PresidentsController($http){
// ...

  function getPresidents(){
    $http
      .get('http://localhost:3000/presidents')
      .then(function(response){
        self.all = response.data.presidents;
    });
  }

  getPresidents();

// ...
}
```

We call `$http`, then our favorite HTTP verb, `.get`. There's one for `.post`, too. It's asynchronous, so we'll use `.then` to make sure when it's _done_ it'll do what we want. And what we want is just to overwrite our `.all` array with the response we get back.

> **Note:** The return value we get from $http.get() is called a *promise*.  A *promise* is an asynchronous operation that hasn't completed yet, but is expected to in the future.  Does that sound like something else in Javascript? 

<!-- Do this for whole class to see-->

Feel free to `console.log(response)` and see everything that comes back. `.data` is just the data, `.presidents` is the key inside our JSON holding an array of presidents.

That's all we're doing in that function. Afterwords, we literally just run the function, which runs when we first load up the app. Easy.

**Now before we move on and you try it yourself, there's an important detail to note.** We've suddenly gone from:

```js
function PresidentsController($http){
  this.all = [];
  // ...
```
to
```js
function PresidentsController($http){
  var self = this;
  self.all = [];
  // ...
```

**Why?** The answer is JavaScript's _scope_. As you've seen in the past few weeks, `this` means different things depending on how many layers deep your code is.

In the previous example, which function is `this` scoped to?

```js
function PresidentsController($http){
// ...

  function getPresidents(){
    $http
      .get('http://localhost:3000/presidents')
      .then(function(response){
        // Where is 'this' scoped to?
        this.all = response.data.presidents;
    });
  }
// ...
}
```

We're 3 functions deep when we call `this.all` – `this` is no longer referring to our controller, it's referring to the function inside `.then`. If you left it that way, you'd never see any data, because to see it in the view, that data needs to be attached directly to our _controller_.

So what's a simple way to make sure we're scoped to the right place? A tiny little variable. The variable you choose is up to you, it's just preference. So if we do:

```js
function PresidentsController($http){
  var self = this;
  self.all = [];
// ...

  function getPresidents(){
    $http
      .get('http://localhost:3000/presidents')
      .then(function(response){
        self.all = response.data.presidents;
    });
  }

  getPresidents();

// ...
}
```

Now we can trust we're talking to the right scope.

Bring this code into your controller, try refreshing your browser, and let's see if it worked!

<img width="752"  src="https://cloud.githubusercontent.com/assets/25366/9017871/7cf4a79e-378e-11e5-85d8-d018f0a7ab21.png">

<details>
  <summary>Describe the difference between $http in Angular and $.ajax in jQuery </summary>
  <p>They both do the same thing: Handle  XMLHttpRequests to make http calls to the web from the client side.</p>
</details>

<!--9:45 25 minutes -->

## Independent Practice

Now that we've got GETing down, it's up to you to try POSTing. Just like any RESTful API, you can add a new president by POSTing to the correct URL. You'll need to modify your controller action to send a new president from the form to our API, and probably look up the Angular documentation to figure out how to do it.

We'll be walking around helping you if you get stuck. In the last few minutes we can see how many people got it!

### Stretch

If you have time, see if you can add the DELETE request as well.

<!--10:10 5 minutes -->

## Conclusion
- How do you inject dependencies into an Angular controller?
- How do you use $http to do a GET request?
- Why did we start using `self` instead of `this`?
- How do you do a POST request with Angular?


## Additional Resources
- [The Docs](https://docs.angularjs.org/api/ng/service/$http)
- [Promises Explained](http://andyshora.com/promises-angularjs-explained-as-cartoon.html)
