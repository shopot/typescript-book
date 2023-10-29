# Why Cypress
Cypress is a great E2E testing tool. Here are a few great reasons to consider it:

* Isolated installation possible.
* Ships with TypeScript definitions out of the box.
* Provides a nice interactive google chrome debug experience. This is very similar to how UI devs mostly work manually.
* Has command - execution seperation which allows for more powerfull debugging and test stability (more on this below).
* Has implicit assertions to provide more meaningful debug experience with less brittle tests (more on this in the tips below).
* Provides the ability to mock out and observe backend XHRs easily without changing your application code (more on this in the tips below).

## Installation

> The steps provided in this installation process will give you a nice `e2e` folder that you can use as boiler plate for your organization. You can just copy paste this `e2e` folder into any existing projects that you want to test with cypress.

Create an e2e directory and install cypress and its dependencies for TypeScript transpiling:

```sh
mkdir e2e
cd e2e
npm init -y
npm install cypress webpack @cypress/webpack-preprocessor typescript ts-loader
```

> Here are a few reasons for creating a separate `e2e` folder especially for cypress: 
* Creating a separate directory or `e2e` makes it easier to isolate its `package.json` dependencies from the rest of your project. This results in less dependency conflicts.
* Testing frameworks have a habit of polluting the global namespace with stuff like `describe` `it` `expect`. It is best to keep the e2e `tsconfig.json` and `node_modules` in this special `e2e` folder to prevent global type definition conflicts.

Setup TypeScript `tsconfig.json` e.g. 

```json
{
  "compilerOptions": {
    "strict": true,
    "sourceMap": true,
    "module": "commonjs",
    "target": "es5",
    "lib": [
      "dom",
      "es6"
    ],
    "jsx": "react",
    "experimentalDecorators": true
  },
  "compileOnSave": false
}
```

Do a first dry run of cypress to prime the cypress folder structure. The Cypress IDE will open. You can close it after you see the welcome message.

```sh
npx cypress open
```

Setup cypress for transpiling typescript by editing `e2e/cypress/plugins/index.js` to match the following:

```js
const wp = require('@cypress/webpack-preprocessor')
module.exports = (on) => {
  const options = {
    webpackOptions: {
      resolve: {
        extensions: [".ts", ".tsx", ".js"]
      },
      module: {
        rules: [
          {
            test: /\.tsx?$/,
            loader: "ts-loader",
            options: { transpileOnly: true }
          }
        ]
      }
    },
  }
  on('file:preprocessor', wp(options))
}
```


Optionally add a few scripts to the `e2e/package.json` file:

```json
  "scripts": {
    "cypress:open": "cypress open",
    "cypress:run": "cypress run"
  },
```

## More description of key Files
Under the `e2e` folder you now have these files: 

* `/cypress.json`: Configure cypress. The default is empty and that is all you need.
* `/cypress` Subfolders: 
    * `/fixtures`: Test fixtures
        * Comes with `example.json`. Feel free to delete it. 
        * You can create simple `.json` files that can be used to provide sample data (aka fixtures) for usage across tests. 
    * `/integration`: All your tests. 
        * Comes with an `examples` folder. You can safely delete it.
        * Name tests with `.spec.ts` e.g. `something.spec.ts`. 
        * Feel free to create tests under subfolders for better organization e.g. `/someFeatureFolder/something.spec.ts`.

## First test 
* create a file `/cypress/integration/first.spec.ts` with the following contents: 

```ts
/// <reference types="cypress"/>

describe('google search', () => {
  it('should work', () => {
    cy.visit('http://www.google.com');
    cy.get('#lst-ib').type('Hello world{enter}')
  });
});
```

## Running in development
Open the cypress IDE using the following command.

```sh
npm run cypress:open
```

And select a test to run.

## Running on a build server

You can run cypress tests in ci mode using the following command.

```sh
npm run cypress:run
```

## Tip: Sharing code between UI and test
Cypress tests are compiled / packed and run in the browser. So feel free to import any project code into your test.

For example you can share Id values between UI and Tests to make sure the CSS selectors don't break:

```js
import { Ids } from '../../../src/app/constants'; 

// Later 
cy.get(`#${Ids.username}`)
  .type('john')
```

## Tip: Creating Page Objects
Creating objects that provide a convenient handle for all the interactions that various tests need to do with a page is a common testing convention. You can create page objects using TypeScript classes with getters and methods e.g. 

```js
import { Ids } from '../../../src/app/constants'; 

class LoginPage {
  visit() {
    cy.visit('/login');
  }
  
  get username() {
    return cy.get(`#${Ids.username}`);
  }
}
const page = new LoginPage();

// Later
page.visit();

page.username.type('john');

```

## Tip: Implicit assertion 
Whenever a cypress command fails you get a nice error (instead of something like `null` with many other frameworks) so you fail quickly and know exactly when a test fails e.g. 

```
cy.get('#foo') 
// If there is no element with id #foo cypress will wait for 4 seconds automatically 
// If still not found you get an error here ^ 


// This \/ will not trigger till an element #foo is found
  .should('have.text', 'something') 
```

## Tip: Explicit assertion 
Cypress ships with quite a few assertion helps for the web e.g. chai-jquery https://docs.cypress.io/guides/references/assertions.html#Chai-jQuery. You use them with `.should` command passing in the chainer as a string e.g.

```
cy.get('#foo') 
  .should('have.text', 'something') 
```

## Tip: Commands and Chaining 
Every function call in a cypress chain is a `command`. The `should` command is an assertion. It is conventional to start distinct *category* of chains and actions seperately e.g. 

```ts
// Don't do this 
cy.get(/**something*/) 
  .should(/**something*/)
  .click()
  .should(/**something*/)
  .get(/**something else*/) 
  .should(/**something*/)

// Prefer seperating the two gets 
cy.get(/**something*/) 
  .should(/**something*/)
  .click()
  .should(/**something*/)

cy.get(/**something else*/) 
  .should(/**something*/)
```

Some other libraries *evaluate and run* the code at the same time. Those libraries force you to have a single chain which can be nightmare to debug with selectors and assertions minggled in. 

Cypress commands are essentially *declarations* to the cypress runtime to execute the commands later. Simple words: Cypress makes it easier. 

## Tip: Using `contains` for easier querying

The following shows an example:

```
cy.get('#foo') 
  // Once #foo is found the following:
  .contains('Submit') 
  // ^ will continue to search for something that has text `Submit` and fail if it times out.
  .click()
  // ^ will trigger a click on the HTML Node that contained the text `Submit`.
```

## Tip: Waiting for an HTTP request
A lot of tests have been traditionally brittle due to all the arbitrary timeouts needed for XHRs that an application makes. `cy.server` makes it easy to 
* create an alias for backend calls
* wait for them to occur

e.g. 

```ts
cy.server()
  .route('POST', 'https://example.com/api/application/load')
  .as('load') // create an alias

// Start test
cy.visit('/')

// wait for the call
cy.wait('@load') 

// Now the data is loaded
```

## Tip: Mocking an HTTP request response
You can also easily mock out a request response using `route`: 
```ts
cy.server()
  .route('POST', 'https://example.com/api/application/load', /* Example payload response */{success:true})
```

## Tip: Mocking time 
You can use `wait` to pause a test for some time e.g. to test an automatic "you are about to be logged out" notification screen:

```ts
cy.visit('/');
cy.wait(waitMilliseconds);
cy.get('#logoutNotification').should('be.visible');
```

However, it is recommended to mock time using `cy.clock` and forwarding time using `cy.tick` e.g. 

```ts
cy.clock();

cy.visit('/');
cy.tick(waitMilliseconds);
cy.get('#logoutNotification').should('be.visible');
```

## Tip: Smart delays and retries
Cypress will automatically wait (and retry) for many async things e.g. 
```
// If there is no request against the `foo` alias cypress will wait for 4 seconds automatically 
cy.wait('@foo') 
// If there is no element with id #foo cypress will wait for 4 seconds automatically and keep retrying
cy.get('#foo')
```
This keeps you from having to constantly add arbitrary timeout (and retry) logic in your test code flow. 


## Tip: Unit testing application code
You can also use cypress to unit test your application code in isolation e.g.

```js
import { once } from '../../../src/app/utils'; 

// Later 
it('should only call function once', () => {
  let called = 0;
  const callMe = once(()=>called++);
  callMe();
  callMe();
  expect(called).to.equal(1);
});
```

## Tip: Mocking in unit testing
If you are unit testing modules in your application you can provide mocks using `cy.stub` e.g. if you want to ensure that `navigate` is called in a function `foo`: 

* `foo.ts`
```ts
import { navigate } from 'takeme';

export function foo() {
  navigate('/foo');
}
```

* You can do this as in `some.spec.ts`: 
```ts
/// <reference types="cypress"/>

import { foo } from '../../../src/app/foo';
import * as takeme from 'takeme';

describe('should work', () => {
  it('should stub it', () => {
    cy.stub(takeme, 'navigate');
    foo();
    expect(takeme.navigate).to.have.been.calledWith('/foo');
  })
});
```

## Tip: Breakpoint
The automatic snapshots + command log generated by the cypress test are great for debugging. That said you can pause test execution if you want. 

First make sure you have chrome developer tools (lovingly called dev tools) open in the test runner (`CMD + ALT + i` on mac / `F12` on windows). Once the dev tools are open you can re-run the test and the dev tools will stay open. If you have the dev tools open, you can pause test execution in two ways:

* Application code breakpoints: Use a `debugger` statement in your application code and the test runner will stop on that just like standard web developement. 
* Test code breakpoints: You can use the `.debug()` command and cypress test execution will stop at it. Alternatively you can use a `debugger` statement in a `.then` command callback to cause a pause. e.g `.then(() => { debugger })`. You can even use it to grab some element `cy.get('#foo').then(($ /* a reference to the dom element */) => { debugger; })` or a network call e.g. `cy.request('https://someurl').then((res /* network response */) => { debugger });`. However idiomatic way is `cy.get('#foo').debug()` and then when the test runner is paused on `debug` you can click on the `get` in the command log to automatically `console.log` any information you might need about the `.get('#foo')` command (and similarly for any other commands you want to debug).


## Resources 
* Website: https://www.cypress.io/
* Write your first cypress test (gives a nice tour of the cypress IDE) : https://docs.cypress.io/guides/getting-started/writing-your-first-test.html
* Setting up a CI environment (e.g. the provided docker image that works out of the box with `cypress run`): https://docs.cypress.io/guides/guides/continuous-integration.html
* Recipes (Lists recipes with descriptions. Click on headings to navigate to the source code for the recipe): https://docs.cypress.io/examples/examples/recipes.html
