Jasmine vs Mocha
================

Testing in JavaScript is becoming expected by developers more and more. But where do you start? There are so many framework choices out there. It can feel pretty overwhelming. This post is a quick overview of the differences between two popular JavaScript testing frameworks: [Jasmine 2](http://jasmine.github.io/) and [Mocha](https://mochajs.org/). We will also discuss commonly used libraries, [Chai](http://chaijs.com/) and [Sinon](http://sinonjs.org/), that are often used in conjunction with Jasmine and Mocha.

## 1. The API

The APIs of Jasmine and Mocha are very similar where you write your test suite with `describe` blocks and each test, also called a spec, using the `it` function.

```js
describe('calculator add()', function() {
  it('should add 2 numbers togoether', function() {
    // assertions here
  });
});
```

The assertions, or expectations as they are often called, are where things start to differ. Mocha does not have a built in assertion library. There are several options though for both Node and the browser: Chai, should.js, expect.js, and better-assert. A lot of developers choose Chai as their assertion library. Because none of these assertion libraries come with Mocha, this is another thing you will need to load into your test harness. Chai comes with three different assertion flavors. It has the `should` style, the `expect` style, and the `assert` style. The `expect` style is similar to what Jasmine provides. For example, if you want to write an expectation that verifies `calculator.add(1, 4)` equals 5, this is how you would do it with both Jasmine and Chai:

__Jasmine__

```js
expect(calculator.add(1, 4)).toEqual(5);
```

__Chai__

```js
expect(calculator.add(1, 4)).to.equal(5);
```

Pretty similar right? If you are switching from Jasmine to Mocha, the path with the easiest learning curve is to use Chai with the `expect` style. In Jasmine, the assertion methods like `toEqual()` use camel case whereas the compliment in Chai uses dot notation, `to.equal()`. Both Jasmine and Mocha use `describe()` and `it()` functions.

## 2. Test Doubles

Test doubles are often compared to stunt doubles, as they replace one object with another for testing purposes, similar to how actors and actresses are replaced with stunt doubles for dangerous action scenes. In Jasmine, test doubles come in the form of spies. A spy is a function that replaces a particular function where you want to control its behavior in a test and record how that function is used during the execution of that test. Some of the things you can do with spies include:

* See how many times a spy was called
* Specify a return value to force your code to go down a certain path
* Tell a spy to throw an error
* See what arguments a spy was called with
* Tell a spy to call the original function (the function it is spying on). By default, a spy will not call the original function.

In Jasmine, you can spy on existing methods like this:

```js
var userSaveSpy = spyOn(User.prototype, 'save');
```

You can also create a spy if you do not have an existing method you want to spy on.

```js
var spy = jasmine.createSpy();
```

In contrast, Mocha does not come with a test double library. Instead, you will need to load in Sinon into your test harness. Sinon is a very powerful test double library and is the equivalent of Jasmine spies with a little more. One thing to note is that Sinon breaks up test doubles into three different categories: [spies](http://sinonjs.org/docs/#spies), [stubs](http://sinonjs.org/docs/#stubs), and [mocks](http://sinonjs.org/docs/#mocks), each with subtle differences.

A spy in Sinon calls through to the method being spied on whereas you have to specify this behavior in Jasmine. For example:

```js
spyOn(user, 'isValid').andCallThrough() // Jasmine
// is equivalent to
sinon.spy(user, 'isValid') // Sinon
```

In your test, the original `user.isValid` would be called.

The next type of test double is a stub, which acts as a controllable replacement. Stubs are similar to the default behavior of Jasmine spies where the original method is not called. For example:

```js
sinon.stub(user, 'isValid').returns(true) // Sinon
// is equivalent to
spyOn(user, 'isValid').andReturns(true) // Jasmine
```

In your code, if `user.isValid` is called during the execution of your tests, the original `user.isValid` would not be called and a fake version of it (the test double) that returns `true` would be used. In Sinon, a stub is a test double built on top of spies, so stubs have the ability to record how the function is being used. 

So to summarize, a spy is a type of test double that records how a function is used. A stub is a type of test double that acts as a controllable replace as well as having the capabilities of a spy.

From my experience, Jasmine spies cover almost everything I need for test doubles so in many situations you won't need to use Sinon if you are using Jasmine, but you can use the two together if you would like. One reason I do use Sinon with Jasmine is for its fake server (more on this later).


## 3. Asynchronous Testing

Asynchronous testing in Jasmine 2.x and Mocha is the same.

```js
it('should resolve with the User object', function(done) {
  var dfd = new $.Deferred();
  var promise = dfd.promise();
  var stub = sinon.stub(User.prototype, 'fetch').returns(promise);

  dfd.resolve({ name: 'David' });

  User.get().then(function(user) {
    expect(user instanceof User).toBe(true);
    done();
  });
});
```

Above, `User` is a constructor function with a static method `get`. Behind the scenes, `get` uses `fetch` which performs the XHR request. I want to assert that when `get` resolves successfully, the resolved value is an instance of `User`. Because I have stubbed out `User.prototype.fetch` to return a pre-resolved promise, no real AJAX request is made. However, this code is still asynchronous.

By simply specifying a parameter in the `it` callback function (I have called it `done` like in the documentation but you can call it whatever you want), the test runner will pass in a function and wait for this function to execute before ending the test. The test will timeout and error if `done` is not called within a certain time limit. This gives you full control on when your tests complete. The above test would work in both Mocha and Jasmine 2.x. 

If you are working with Jasmine 1.3, asynchronous testing was not so pretty.

__Example Jasmine 1.3 Asynchronous Test__

```js
it('should resolve with the User object', function() {
  var flag = false;
  var david;

  runs(function() {
    var dfd = new $.Deferred();
    var promise = dfd.promise();

    dfd.resolve({ name: 'David' });
    spyOn(User.prototype, 'fetch').andReturn(promise);

    User.get().then(function(user) {
      flag = true;
      david = user;
    });
  });

  waitsFor(function() {
    return flag;
  }, 'get should resolve with the model', 500);

  runs(function() {
    expect(david instanceof User).toBe(true);
  });
});
```

In this Jasmine 1.3 asynchronous test example, Jasmine will wait a maximum of 500 milliseconds for the asynchronous operation to complete. Otherwise, the test will fail. `waitsFor()` is constantly checking to see if `flag` becomes true. Once it does, it will continue to run the next `runs()` block where I have my assertion.


## 4. Sinon Fake Server

One feature that Sinon has that Jasmine does not is a fake server. This allows you to setup fake responses to AJAX requests made for certain URLs.

```js
it('should return a collection object containing all users', function(done) {
  var server = sinon.fakeServer.create();
  server.respondWith("GET", "/users", [
    200,
    { "Content-Type": "application/json" },
    '[{ "id": 1, "name": "Gwen" },  { "id": 2, "name": "John" }]'
  ]);

  Users.all().done(function(collection) {
    expect(collection.toJSON()).to.eql([
      { id: 1, name: "Gwen" },
      { id: 2, name: "John" }
    ]);

    done();
  });

  server.respond();
  server.restore();
});
```

In the above example, if a `GET` request is made to `/users`, a 200 response containing two users, Gwen and John, will be returned. This can be really handy for a few reasons. First, it allows you to test your code that makes AJAX calls regardless of which AJAX library you are using. Second, you may want to test a function that makes an AJAX call and does some preprocessing on the response before the promise resolves. Third, maybe there are several responses that can be returned based on if the request succeeds or fails such as a successful credit card charge, an invalid credit card number, an expired card, an invalid CVC, etc. You get the idea. If you have worked with Angular, Sinon's fake server is similar to the _$httpBackend_ service provided in angular mocks.

## 5. Running Tests

Mocha comes with a command line utility that you can use to run tests. For example:

```bash
mocha tests --recursive --watch
```

This assumes your tests are located in a directory called `tests`. The recursive flag will find all files in subdirectories, and the watch flag will watch all your source and test files and rerun the tests when they change.

Jasmine however does not have a command line utility to run tests. There are test runners out there for Jasmine, and a very popular one is [Karma](http://karma-runner.github.io/) by the Angular team. Karma also allows support for Mocha if you'd like to run your Mocha tests that way.

## Summary

In conclusion, the Jasmine framework has almost everything built into it including assertions/expectations and test double utilities (which come in the form of spies). However, it does not have a test runner so you will need to use a tool like Karma for that. Mocha on the other hand includes a test runner and an API for setting up your test suite but does not include assertion and test double utilities. There are several choices for assertions when using Mocha, and Chai tends to be the most popular. Test doubles in Mocha also requires another library, and Sinon.js is often the de-facto choice. Sinon can also be a great addition to your test harness for its fake server implementation.

So, if you were to choose a test framework setup today, what might it look like?

If you go with Jasmine, you will likely use:

* Karma (for the test runner)
* Sinon (possibly for its fake server unless your framework provides an equivalent, like `$httpBackend` if you are using Angular)

If you go with Mocha, you will likely use:

* Chai (for assertions)
* Sinon (for test doubles and its fake server)
* Karma or `mocha` CLI (for the test runner)

Trying to figure out testing libraries/frameworks to use for JavaScript can be tough but hopefully this article has made it more 
clear as to what some of the main differences are between Jasmine and Mocha. You can't really go wrong with either choice. The important thing is that you are testing!
