# Testing Guidelines

## Libraries/Frameworks
* Runners
  * Mocha
  * Karma
* Assertion Libraries
  * Chai
  * Sinon
* Mocking Libraries
  * fetch-mock
  * nock
  * proxyquire


## Writing good tests

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">[on my death bed, struggling to speak]<br><br>me: if..your..unit..test..makes..a..network..call..it&#39;s..not..a..unit..test..<br><br>[dies]</p>&mdash; Rebecca Slatkin (@RebeccaSlatkin) <a href="https://twitter.com/RebeccaSlatkin/status/852627030092939274">April 13, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Unit Test: Successful. Integration Test: Not performed. <a href="https://t.co/eabqqTaw7X">pic.twitter.com/eabqqTaw7X</a></p>&mdash; Mohammed (@Dadash91) <a href="https://twitter.com/Dadash91/status/831317037037285376">February 14, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

#### Should have clear Arrange, Act & Assert steps
Prefer clarity over DRY-ness. It's annoying to have to move around a test file to understand what the part of the arrange step in a test is. If you are using stubs, unless they return the same thing for every test (they likely shouldn't do,) define what your stubs return inside the `it` block so that, as the reader it's clear what the set up was and how each part of the test is expected to behave.

```js
describe('Some Module', () => {
  it('should do something', () => {
    // ARRANGE

    // ACT

    // ASSERT
  });
});
```

This example has clear Arrange, Act & Assert steps
```js
it('should call first controller when flag exists in res.locals.flags', () => {
  // ARRANGE
  const req = {};
  const res = { locals:{ flags:{ testFlag: true } } };
  const controller1 = sinon.stub();
  const controller2 = sinon.stub();
  const subject = varyControllerOnFlag('testFlag', controller1, controller2);

  // ACT
  subject(req, res);

  // ASSERT
  expect(controller1).to.have.been.called.once;
  expect(controller2).to.not.have.been.called;
});
```

It may be that there is now test setup to be done. That is fine but an Act and Assert step will always be required. It also maybe that the Assert step happens within a `.then` block that is chained to your Action.

#### Should avoid too many assertions in one test
Sometimes it is unavoidable but be aware that if you have too many `expect(...)`s within a test you are likely testing too many things at once and you could consider whether some of them could be covered by separate tests or if they match the test's description.

#### Should act as documentation
Don't be afraid to nest `describe` or `context` blocks in order to provide more clarity when testing a given scenario.

```
Alerts Read controller
  when `preferred/preferences` call fails
    ✓ should call next
  when `followed/concepts` call fails
    ✓ should call next
  when `user/<uuid>` fails
    with a 500
      ✓ should add `userData: undefined` to the viewModel
    with bad data
      ✓ should add `userData: undefined` to the viewModel
  when `user/<uuid>/lists` fails
    with a 404
      ✓ should add `userMissingInEmailPlatform` property to the newsletters object
    with a 500
      ✓ should add `userMissingInEmailPlatform` property to the newsletters object
  when everything returns 200s
    ✓ should call res.render with the correct viewModel
```

Because of nesting tests within `describe` blocks we can clearly see here what the expected behaviour of the `Alerts Read controller` is given different scenarios. This is often more prevelant with integration tests rather than unit tests but don't be afraid to add more context in order to aid someone else with understanding what should happen.


#### Should have NO conditionals
There should be no conditionals (for, ifs or whiles) in your tests. _You_ should define what happens, not the computer.

#### Should fail at some point (Whaaat?)
Seriously. You _**must**_ see your test fail at some point so that you know it works. This is clearer when practicing TDD as no production code should've been written and it should fail but if you write the tests after writing the actual code, check the test fails by changing some of the production code (and then changing it back obviously)!

```js
// This test should clearly fail right? Expect false to be true?

it('should fail', () => {
	Promise.resolve().then(() => {
		expect(false).to.be.true;
	});
});

// "Haha, what a terrible example Simon" I hear you say.
```
```js
Testing examples
  ✓ should fail

1 passing (9ms)

// Oh....
```

Oh, indeed. What has happened here is that because this is an asyncronous test we either need to pass in a `done` callback to our test function or `return` the Promise from the test. We prefer the latter

```js
// Using the done callback.
it('should fail', (done) => {
	Promise.resolve().then(() => {
		expect(false).to.be.true;
		done();
	});
});

// `done` callback version fails with the unhelpful error:
// `Error: Timeout of 2000ms exceeded. For async tests and hooks, ensure "done()" is called; if returning a Promise, ensure it resolves.`


// Returning the Promise. (Preferred)
it('should fail', () => {
	return Promise.resolve().then(() => {
		expect(false).to.be.true;
	});
});

// This now fails with, the much nicer, `AssertionError: expected false to be true`🙌

// Or with the fancy new async/await syntax:
it('should fail', async () => {
    const result = await Promise.resolve(false)
    expect(result).to.be.true
})
```

This is why you need to make sure your test fails. You would've caught this crap test.


* What to test
* What not to test
* Difference between unit and integration tests
  * What should be mocked in each type
* Common signs of poor testing
* Test messages/descriptions
* Arrange, Act, Assert


## Test Coverage

*  How it works
*  Don't rely on it
*  It is not a product goal!
*  Setting up coverage in an app

## Resources

* [Unit Tests, How to Write Testable Code and Why it Matters][1] by Sergey Kolodiy
* [Writing Beautiful JavaScript Tests][2] by Kim Joar Bekkelund

[1]: https://www.toptal.com/qa/how-to-write-testable-code-and-why-it-matters "Unit Tests, How to Write Testable Code and Why it Matters"
[2]: https://speakerdeck.com/kimjoar/writing-beautiful-javascript-tests "Writing Beautiful JavaScript Tests"
