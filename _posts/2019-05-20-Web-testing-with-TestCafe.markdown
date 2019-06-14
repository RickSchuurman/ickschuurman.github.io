---
layout: post
title:  "Web testing with TestCafe"
date:   2019-05-20
categories: Web-testing
tags: TestCafe Web-testing
excerpt: Web testing with TestCafe
---

* content
{:toc}

## Testcafe

TestCafe is a node.js tool to automate end-to-end web testing. You can write tests in JS or TypeScript, run them and view results.
TestCafe runs on Windows, MacOS, and Linux. It supports desktop, mobile, remote and cloud browsers (UI or headless). Its free and open source

TestCafe works by serving the test site via a proxy server. The server injects scripts into the page which can inspect and control elements on the page.

[Link](https://devexpress.github.io/testcafe/) to TestCafe homepage.


## Set up


Prerequisite: Node and NPM should be installed

Install TestCafe with the command below

  ``
   $ npm install testcafe --save-dev 
  ``

## Structure test code

* Import the testcafe module

     ````javascript
     import { Selector } from 'testcafe';
     ````
     
* Testcafe tests must be organized into categories. We can do this by declaring categories using the fixture function. A file can contain one or more fixtures.
    
    ````javascript
    fixture `Creating first fixture`
    ````
    
* Specify a start page for the fixture using the page function

  ````javascript
    fixture `Creating first fixture`
        .page `https://www.joyoftesting.nl/`;
    ````

* Create a test by using the test function

  ````javascript
    fixture `Creating first fixture`
        .page `https://www.joyoftesting.nl/`;
    
    test ('Creating first test', async t => {
      // Test code  
    });
    ````


## Run your first test

Run your first test from the command line where you specify the target browser and file path


  ``
  $ testcafe chrome system-test/first-test.js
  ``

The browser will start ran your test    

 ````bash
fixture `Creating first fixture`
 Running tests in:
  - Chrome 74.0.3729 / Mac OS X 10.14.5
 
  Creating first fixture
  ✓ Creating first test
 
 
  1 passed (1s)
````

## Run test in Live mode / Develop mode

Live mode ensures the TestCafe process and browsers remain active while you work on tests you can see test results instantly because the tests are restarted when you make changes. 

When you run tests with live mode enabled, TestCafe opens the browsers, runs the tests, shows the reports, and waits for further actions.
Then TestCafe starts watching for changes in the test files and all files referenced in them (like page objects or helper modules). Once you make changes in any of those files and save them, TestCafe immediately reruns the tests.
When the tests are done, browsers stay on the last opened page so you can work with it and explore it with the browser's developer tools.
You can use live mode with any browsers: local, remote, mobile or headless.


Run test with live mode enabled with the L option

``
$ testcafe chrome system-test/first-test.js -L
``
    
 ````bash
Live mode is enabled.
TestCafe now watches source files and reruns
the tests once the changes are saved.

You can use the following keys in the terminal:
'Ctrl+S' - stops the test run;
'Ctrl+R' - restarts the test run;
'Ctrl+W' - enables/disables watching files;
'Ctrl+C' - quits live mode and closes the browsers.


Watching the following files:
  /Users/rickschuurman/Workspace/testcafe-framework/system-test/test.js
 Running tests in:
 - Chrome 74.0.3729 / Mac OS X 10.14.5

 Creating first fixture
 ✓ Creating first test


 1 passed (1s)

Make changes to the source files or press Ctrl+R to restart the test run.
````

[Link](https://devexpress.github.io/testcafe/documentation/using-testcafe/common-concepts/live-mode.html) to TestCafe Live Mode.


## Automate our own test scenario

For this guide we use the MEAN example app which I created for this purpose. You can find it here:

``
https://github.com/RickSchuurman/mean-example-app
``

* As a user <BR />
* When I'm on the My Message Page <BR />
* I can click on the first post <BR />
* And validate the content <BR />

### Selectors

Selector is a function that identifies a webpage element in the test. The selector API provides methods and properties to select elements on the page and get their state.
TestCafe uses standard CSS selectors to locate elements. It’s like when you use document.querySelector. 

[Link](https://devexpress.github.io/testcafe/documentation/test-api/selecting-page-elements/) to TestCafe Selectors.



  ````javascript
        import {Selector} from 'testcafe'; // import Selector
        
        const post = Selector('.mat-expansion-panel').nth(0); // Select the first post
        
        fixture('Test MEAN example app')
            .page('http://localhost:4200'); // Starting page
        
        
        test('Validate the first post content on the My Message page', async t => {
            // test code
        });
  ````

### Actions 

Test API provides a set of actions that enable you to interact with the webpage. Actions are implemented as methods in the test controller object and can be chained.

In this example we will use the Click action

[Link](https://devexpress.github.io/testcafe/documentation/test-api/actions/) to TestCafe Actions.



  ````javascript
        import {Selector} from 'testcafe'; // import Selector
        
        const post = Selector('.mat-expansion-panel').nth(0); // Select the first post
        
        fixture('Test MEAN example app')
            .page('http://localhost:4200'); // Starting page
        
        
        test('Validate the first post content on the My Message page', async t => {
            await t.click(post); // click the first post
        });
  ````
  
  
### Assertions 

We now need assertions to check data or behaviors. The test controller provides an expect function that should check the result of performed actions.
In our test we check that the post content contains the text "Test content"

[Link](https://devexpress.github.io/testcafe/documentation/test-api/assertions/) to TestCafe Assertions.



  ````javascript
        import {Selector} from 'testcafe'; // import Selector
        
        const post = Selector('.mat-expansion-panel').nth(0); // Select the first post
        const postContent = Selector('.mat-expansion-panel-content').nth(0); // Select the first post content
        
        fixture('Test MEAN example app')
            .page('http://localhost:4200'); // Starting page
        
        
        test('Validate the first post content on the My Message page', async t => {
            await t.click(post); // click the first post
            await t.expect(postContent.innerText).eql('Test content!'); // validate post content
        });
  ````

  


## Pros and Cons

| Pros                                  | Cons                                                                                      |
|---------------------------------------|----------------:|
| Easy setup                            |                 |
| Live mode / Develop mode              |                 |
| Great documentation                   |                 |
| Cross Browser Testing                 |                 |

