name: E2E testing
class: middle, center, title

# E2E Testing with Protractor and&nbsp;Friends
## or: How I Learned to Stop Worrying and Test the DOM
### Ryan Parsley
#### October 25, 2019

???

Hello, I'm Ryan Parsley. I'm a Sr Software Engineer working on Carrier 360.

I have the privilege to teach you a thing that is otherwise a little tricky to learn.

It's not a very tricky subject, I don't want to scare you into a defensive posture.

It's just that best practices feel more rarely discussed with E2E testing than they do with Unit testing. 

---
background-image: url(./assets/testPyramid.png)
background-size: 500px auto

# The code confidence parfait

???

I'm getting ahead of myself. Let's start with answering "What is an end to end test?"

---

# What is an end to end test!?

* Test suite with a browser dependency
* AKA: Integration test
* AKA: Functional test

???

There are a lot of tools to provide you code confidence, and many of those tools come by more than one name. Or, are otherwise hard to distinguish between. I'm calling this class of tests e2e tests because that's the name of the folder the angular cli puts them in by default.

---
class: middle, center

# In other words...
## E2E tests run in the browser to more closely match real world user interaction with our app.
---

background-image: url(./assets/whyE2E.gif)
background-size: 500px auto

# But we have unit tests...

???

The whole isn't always equal to the sum of it's parts.

---

# Key components of end to end testing

--
* Browser

--
* Tool to drive a browser
  * Selenium/WebDriver
  * Puppeteer

--
* Test Framework (interaction)
  * Protractor
  * Cypress

--
* Test Framework (assertions)
  * Jasmine
  * Mocha
  * QUnit

???

Basically, you have a real browser somehow controlled by code to poke around your website. If this makes sense, you're way ahead of the game already. We have a framework that negotiates the interaction through a driver that controls a browser to satisfy assertions.

Protractor speaks Jasmine and adds Angular sugar on top of WebDriver. It communicates with a browser via the Selenium Server.

---
class: middle, center, title

# Protractor

---

# So what is Protractor?

--

> Protractor is an end-to-end test framework for Angular and AngularJS
applications. Protractor runs tests against your application running in a real
browser, interacting with it as a user would. <span>&mdash;[Protractortest.org]()</span>

???

In some ways, it's easier to get started with E2E testing than with unit testing.

With unit testing, you purposefully break out of standard app flow and Dependency Injections to focus one each piece of the machine. With E2E testing, you're taking a step back to watch the machine work. So you don't need to wire up any Dependency injection or manually trigger lifecycle events. You're testing the fully functioning app. 

---
# What does that look like!?

```javascript
// Starts out pretty familiar
describe('todo list', function() {
  it('should add a todo', function() {

    // We don't have this in Unit tests
    browser.get('https://angularjs.org');

    // click on a DOM element
    element(by.css('[value="add"]')).click();

    // Locate something in the DOM to assert against
    var todoList = element.all(by.repeater('todo in todoList.todos'));
    expect(todoList.count()).toEqual(3);
    ...
  });
});
```

???

Describe and it should look familiar to you if you've done unit testing. Those
are provided for us to group out specs and associate human readable labels
should they fail.

Browser is a global that operates as a conduit to the driver. This is a wrapper
around Selenium's WebDriver.

Element is a function that takes a locator and returns an ElementFinder Object.
You can chain actions such as `click()` to the results to interact with the
ElementFinder.

Don't get hung up on ElementFinder as a thing. Suffice it to say that you need a
reference to dom elements, and  `element()` is what affords you that.

element.all() is there for when you need to return a collection of DOM elements.

---
class: middle, center

# Good news

## Angular CLI wires up Protractor out of the box...

---
class: middle, center

# Bad news

--

## ...kind of
![welcome to protractor](./assets/welcomeToProtractor.png)

???
Selenium has comically useless feedback sometimes. Unfortunately, the first opportunity for this is the first time you ever use it if chrome versions come out of sync with node module dependency. If you see something like this, upgrade chrome.

---

class: middle, center, title

# Less talk more code

---

# App to be tested
## AcuteBlog

* Authentication
  * Register
  * Login
  * Restricted access
* Post
  * Create (logged in users)
  * Read (anyone)
  * Update (only your own)
  * Delete (only your own)

---

# Exercise 1

* Clone [the repo](https://jbhunt.visualstudio.com/Training/_git/e2e-acuteBlog)
  and install packages 
```
npm i
```
* Run the app
```
npm run start
```
* Run the e2e tests
```
npm run e2e -- --suite=ex1
```
* Write a test that confirms the app is running

---
background-image: url(./assets/takeThis.jpg)
background-size: 500px auto

---

# element()

Global function that takes a locator and returns a 'ElementFinder' object

* getText()
* click()
* sendKeys()
* clear()
* ...

???

Let's not over think this. You assert against the state of the dom and this is
how you get a reference to such elements. The important think to think about are
the methods you can use.

* getText()
* click()
* sendKeys()
* clear()

Clicking links, inspecting text, and filling out forms is the majority of what
you're going to be doing, but check the docs for the rest of the api when you
need more.

---

# Locators
## Be as specific as you need, but no more

```javascript
element(by.css('.foo'));
```

* buttonText
* cssContainingText
* linkText

???

This is another concept that's largely provided by webdriver. In the days of
AngularJS, there seemed to be more work in custom locators for ngModel, binding,
and other angular centric stuff. That doesn't seem to be as much of a focus now.

And that's fine! Most of what you need can be achieved with `by.css`.

Do not simply copy the class attribute from chrome inspector and past that in
`by.css()`. That is often littered with presentational helpers that introduce
additional brittleness to your tests. If someone comes by and eliminates a
`.float` class either because the element's position has changed or a good old
fashion CSS refactor, that should not likely make a test fail.

---

# Exercise 2
## Happy path of an anonymous reader

--

* Home page should be accessible
--

* User list page should *not* be accessible
--

* Bogus links redirect to 404
--

* Posts should be readable
--

???
Now that you're a little better equipped, let's tackle a few more tests.

---

class: middle, center, title

# Tactics

???

Up until now I send you out there with the basic building blocks of E2E testing
and you've got a lot done. Hopefully by the time you're done with Exercise 2
you're thinking about drying up your tests and itching for tools that will let
you feel more clever all in all.

---

# Page object

A plain JavaScript object that encapsulates the properties of a template

???

Sounds fancy? We're really just wiring up easy ways to get at the things we need
to access multiple times. This is a widely adopted pattern to essentially define
an adhoc API for interacting with things on a given route.

--

### Defined in the page object file...

```javascript
// ./pages/somepage.po.ts
var somePage = function(){
  this.foo = element(by.css('.foo'));
}
```

### Used in a test...

```javascript
// ./specs/foo.spec.ts
somePage.foo.sendKeys('bar');
```

---
background-image: url(./assets/slowTests.jpg)
background-size: 400px auto

???

Do not sleep/ wait for static amounts of time.

Learn more about what you're waiting for and resolve appropriately.

---

# When you need to wait for things
## Do not do this

```javascript
public async verifySizeAfterResize(): Promise<boolean> {
  // Vote of no confidence in DOM readiness
  await this.actions.shortWait('Short Wait');
  const width = 1000, height = 800;
  browser.driver.manage().window().setSize(width, height);
  // ಠ_ಠ
  await this.actions.mediumWait('Medium Wait');
  await this.actions.moveMouseToElement(
    this.myLoadsSearchIcon,
    'Move mouse to Icon');
  await this.actions.click(
    this.myLoadsSearchIcon,
    'Click the search icon');
  // ಠ_ಠ
  await this.actions.shortWait('Short Wait');
  return await this.actions.isElementDisplayed(
    this.searchButtonSearchPanel,
    'Display the Search Button');
}
```

???

When you set a static wait time, you guarantee the tests will wait an amount of
time that you hope is greater than the worst case scenario. Craig searched the
specs and calculated about 28 minutes of waiting is hard coded.

---

# A better approach
## Resolve promises instead of choosing arbitrary times

```
beforeEach(async () => {
  await myLoads.open('/my-loads?doAsUserID=JBHKOPA2');
  await pageStable();
});

const pageStable = async() => {
  Promise.all([
    await browser.wait(EC.not(EC.visibilityOf(myLoads.loadingMask)), 20000),
    await myLoads.actions.waitUntilElementInvisible(
      myLoads.growl,
      'Wait for growl to go away')
  ]);
};

it('should open my-loads #smoke', () => {
  expect(myLoads.title.getText()).toEqual(titleText);
});

```

---
# ExpectedConditions

A library of canned expected conditions that are useful for protractor.

Each condition returns a function that evaluates to a promise.

```javascript
var EC = protractor.ExpectedConditions;
var button = $('#xyz');
var isClickable = EC.elementToBeClickable(button);

browser.get(URL);
browser.wait(isClickable, 5000); //wait for an element to become clickable
button.click();
```

[Documentation](https://www.protractortest.org/#/api?view=ProtractorExpectedConditions)

???

The documentation calls out that ExpectedConditions are particularly useful for
non-angular apps. I don't feel that's accurate. You're going to need them.

---
background-image: url(./assets/refactor.jpg)
background-size: 500px auto

???

You've already visited a route and made assertions, but now you have a new tool
in your belt, so let's leverage a page object to similar effect. It's time
for...

---

# Exercise 3
## Users should be able to register
### Interact with forms and leverage expected conditions

--

* Leverage a Signup Page object
--

* Signup Route should be accessible
--

* Form should function
--

* Leverage `ExpectedConditions` to wait for the alert upon submission

???

This should feel familiar at this point but lets use our new tool the Page
Object to make interacting with this page less brittle.

---
background-image: url(./assets/abstraction.jpg)
background-size: 500px auto

# Helper Methods

???

Page objects aren't the only way to share between tests. It's more appropriate
to write some simple javascript helpers for certain tasks.

* Login
* Logout
* State setup (seed a database)

---
# While it's cool we can do this...

```javascript
// in any given spec with respect to any given route that needs to care

browser.executeScript('return window.localStorage.getItem("user");');
```
--

## It's more humane to do it like this...
```javascript
// ./utils/helpers.ts

export function getCurrentUser() {...}
```
---
# Helper: Login()

```javascript
export function login() {
   // Navigate to a login page
   loginPage.navigateTo();
   
   // fill out the login form
   loginPage.submitLogin('foo', 'bar');
   
   // any extra steps to get us to a stable logged in state
   ...
}
```

---
# Exercise 4
## Authenticate a user

* Leverage helper utilities for login and logout
* Write a test that logs the user in
* Write an assertion that proves the user is logged in
* Visit an authenticated route (/users)
* Make an assertion about the user list
---

class: middle, center, title

# Strategy

---
background-image: url(./assets/should.gif)
background-size: 500px auto

# What should we test?

???

End to end tests should focus on actions more than what is rendered on the screen. We will use what's on the screen to assert if action complete correctly, but the content rendered to the page is more the per view of the unit tests around that functionality.

Example: when testing if clicking a link takes you to an appropriate page, you should not be concerned with if that page has specific content if you can avoid it. Choosing the url, or active menu item are likely better things to assert against.

---
class: middle, center

# Workflows

???

Tests should not be named or otherwise organized in a way that is coupled with a
bug or task. E2E tests are meant to hang around for a long time to tell the
story of what a user should experience as they perform various tasks. They will
likely hit many components and multiple routes so naming after a given route is
largely not what we want to do.

---
class: middle, center

# Smoke tests

???

When we speak of smoke tests, we're largely referring to a subset of our tests
that are read only and can be run against the production environment without
fear of changing prod data. We'll tag these tests `#smoke` (eg `it('should visit
my-loads #smoke', () => {...})`)

---
background-image: url(./assets/codeConfidence.jpg)
background-size: 350px auto


# Change is the only constant in E2E

???

It's inevitable that things will change on the frontend. The DOM will change as
we redesign or add features. The DOM will change as we _refactor_ CSS. Content
will change over time. If these things weren't true, we wouldn't need to do
regression testing. These changes are part of the reason that E2E testing is the
most brittle of testing paradigms. Be aware of this brittleness and avoid it as
much as you can.

---
class: middle, center

# Maintainablility

???

In theory, refactoring your code should not need you to update a spec, because
the spec is about how your app works. Refactoring only affects implementation
details. Abstracting these to the page object enables this separation of intent
and implementation.

---

# Do not...
* use overly specific selectors to find elements (don't just copy the class values from chrome inspector).
* assert against copy that's likely to change. (Switching copy from using Oxford Comma to AP Style should not break a test).

---

# Do...
  * introduce unique data attributes (uncoupled from styling) if you don't otherwise have a reasonable selector to hook on to

---

class: middle, center, title

# Anti-patterns

---

# Timeouts

```javascript
// DO NOT DO THIS!
browser.sleep(20000)
```

---

# Brittle selectors
```Javascript
//(╯°□°)╯︵ ┻━┻
originStateSearchBox = element.all(by.className(
  'ui-inputtext ui-widget ui-state-default ui-corner-all' +
  'ui-autocomplete-input ng-star-inserted'
)).get(1);
```
---

background-image: url(./assets/noTests.jpg)
background-size: 500px auto

#  Overly eager testing
# 

---
# Exercise 5
## Happy path of a user with an account

--

* Add Post button exists for authenticated users only
--

* Logged in users can create posts
--

* Logged in users can edit posts
--

---
class: links

# Resources

* [Code Repo](https://jbhunt.visualstudio.com/Training/_git/e2e-acuteBlog)
* [Presentation](https://ryanparsley.github.io/angularE2E/#1)
* [Presentation Source](https://github.com/RyanParsley/angularE2E)

---
class: links

# Further reading

* [https://martinfowler.com/bliki/TestPyramid.html](https://martinfowler.com/bliki/TestPyramid.html)
* [https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html)
* [https://kentcdodds.com/blog/unit-vs-integration-vs-e2e-tests](https://kentcdodds.com/blog/unit-vs-integration-vs-e2e-tests)
* [https://www.cypress.io/blog/2019/08/02/guest-post-angular-adding-cypress-ui-tests-to-your-devops-pipeline/](https://www.cypress.io/blog/2019/08/02/guest-post-angular-adding-cypress-ui-tests-to-your-devops-pipeline/)
* [https://moduscreate.com/blog/protractor-and-jasmine-data-provider-write-once-test-many/](https://moduscreate.com/blog/protractor-and-jasmine-data-provider-write-once-test-many/)
* [Protractor's Official Typescript Examples](https://github.com/angular/protractor/tree/5.4.1/exampleTypescript)
* [Code Repo From Testing Angular Applications](https://github.com/testing-angular-applications/testing-angular-applications/tree/master/chapter09)
* [https://www.blackpepper.co.uk/blog/if-youre-using-browser-sleep-in-your-cucumber-protractor-selenium-tests-youre-doing-it-wrong](https://www.blackpepper.co.uk/blog/if-youre-using-browser-sleep-in-your-cucumber-protractor-selenium-tests-youre-doing-it-wrong)
