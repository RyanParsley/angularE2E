name: E2E testing
class: middle, center, title

# E2E testing with Protractor and&nbsp;Friends
## or: How I Learned to Stop Worrying and Test the DOM
### Ryan Parsley
#### October 25, 2019

???

Hello, I'm Ryan Parsley. I'm a Sr Software Engineer working on Carrier 360.

I have the privilege to teach you a thing that is otherwise a little tricky to learn.

It's not a very tricky subject, I don't want to scare you into a defensive posture.

It's just that best practices feel more rarely discussed with E2E testing than they do with Unit testing. 

---

# What is an end to end test!?

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

# Tests that run in the browser to more closely match user interaction with our app.

> Protractor is an end-to-end test framework for Angular and AngularJS
applications. Protractor runs tests against your application running in a real
browser, interacting with it as a user would. <span>&mdash;[Protractortest.org]()</span>

???

In some ways, it's easier to get started with E2E testing than with unit testing.

With unit testing, you purposefully break out of standard app flow and Dependency Injections to focus one each piece of the machine. With E2E testing, you're taking a step back to watch the machine work. So you don't need to wire up any Dependency injection or manually trigger lifecycle events. You're testing the fully functioning app. 

---
# Key components of end to end testing
* Test Framework (interaction)
  * Protractor
  * Cypress
* Test Framework (assertions)
  * Jasmine
  * Mocha
  * QUnit
* Tool to drive a browser
  * Selenium/WebDriver
  * Puppeteer
* Browser

???

Basically, you have a real browser somehow controlled by code to poke around your website. If this makes sense, you're way ahead of the game already. We have a framework that negotiates the interaction through a driver that controls a browser to satisfy assertions.

By default, Angular ships with
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

---
class: middle, center

# Good news: Angular CLI wires up Protractor out of the box

---
background-image: url(./assets/welcomeToProtractor.png)
background-size: contain

# Bad news: ...kind of

???
Selenium has comically useless feedback sometimes. Unfortunately, the first opportunity for this is the first time you ever use it if chrome versions come out of sync with node module dependency. If you see something like this, upgrade chrome.

---
# Links
