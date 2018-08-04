## Introduction
While the [WebDriver](http://docs.wpm.neustar.biz/testscript-api/biz/neustar/wpm/api/WebDriver.html) and [HttpClient](http://docs.wpm.neustar.biz/testscript-api/biz/neustar/wpm/api/HttpClient.html) interfaces provide WPM with the automation routines for controlling an agent (browser in the case of Selenium and simple HTTP client with HttpClient) they do not provide a mechanism for measuring performance and segmenting that performance into logical steps and transactions.  This functionality is provided by Neustar's [Test](http://docs.wpm.neustar.biz/testscript-api/biz/neustar/wpm/api/Test.html) interface which offers a number of different methods that can be helpful in both performance monitoring and load testing.  In today's post I will just be covering the basic components needed to group page load time metrics into logical steps and transactions. 

## Getting Your Client
The Test interface provides access to the two client types that you will use to automate actions, WebDriver and HttpClient.  WebDriver is a framework that allows you to control a browser and interact with a web application just as an end-user would.  This approach gives the best visibility into an applications performance and makes scripting easier because the Browser does all the hard work for you.  The browser will execute JavaScript, render CSS, and handle rewriting URLs that may be requested as part of a form submission.  WPM supports both Chrome and Firefox for real browser (RBU) monitoring.  In order to instantiate a webDriver object you will need to have the following call at the top of your script (this is almost always the first line in your script):

```javascript
var driver = test.openBrowser();
```

When writing a script for a load test you will need to specify a browser type as an argument to the *openBrowser* method.  You can specify *CHROME* for Chrome or *FF* for Firefox.  For performance monitoring the browser type is a configuration of the monitoring service and thus does not need to be explicitly set in the monitoring script.

The HttpClient on the other hand is a simple interface that emulates various HTTP verbs (ex: POST, GET) and doesn't execute JavaScript nor does it render CSS or do any work that a browser might do when rendering pages or processing requests.  You will see HttpClient based scripts often referred to as Virtual Users (VU).  HttpClient does not provide as accurate a performance metric and scripts that use it are generally more complicated and require more time to develop.  To instantiate an HttpClient object you will need to have the following line of code at (or near) the top of your script:

```javascript
var client = test.getHttpClient();
```

In addition to acting as a VU for the WPM platform the HttpClient also provides access to some features that allow you to interact with the proxy that sits infront of all requests that are made by the WPM platform.  This allows you to do things like set SSL Certificate versions and modify HTTP request/response headers.  You can also get access to an instance of HttpClient from the webDriver interface as follows:

```javascript
var driver = test.openBrowser();
var client = driver.getHttpClient();
```

This will allow you to intercept/rewrite headers and perform other tasks that can benefit your RBU test.  In the end, the goal is to get a client that you can automate for performance monitoring.  A good way to determine which client type to instantiate is:

- **HttpClient if:** 
  - You want to do basic tests of GET/POST performance & availability.
  - You want to test an API or other endpoint that only returns text.
  - You just want to measure availability of a page.
- **WebDriver + HttpClient if:**
  - You want to monitor complex applications that rely on client-side code.
  - You want to simplify script development.
  - You want to have control over the WPM proxy that will make the requests.

For real browser scripts it almost always makes sense to grab instances of both the webDriver and the HttpClient.

## Establish A Transaction
Alright, we're ready to go...we've got our client, maybe we've made some modifications to the WPM proxy using HttpClient, now we want to define our transaction.  This part is super easy; all actions that will generate performance metrics and need to be consider as part of the transaction will be defined inside the function argument to the *beginTransaction* method:

```javascript
test.beginTransaction(function() { 
  // Our code will go here.
});
```

In JavaScript functions are first-class objects so they can be passed as parameters to another function, they are also often defined anonymously and inline as I've done above.  This can be confusing and unfamiliar if you come from a background in C/C++, Python or other procedural languages.  So, there is an alternative design pattern you can use:

```javascript
test.beginTransaction();
// Our code will go here.
test.endTransaction();
```

This approach is often easier for beginners.  Either approach will work and I will most likely use the first approach in future blog posts.

One final comment about transactions, any actions that you want to be counted towards the transaction load time needs to appear *inside* the transaction definition.  Doesn't matter if that's using the first approach (in the function body of the anonymous function) or the second approach (between the *beginTransaction* and *endTransaction* calls).  You can put page/URL requests outside of the transaction definition but the load time they generate will not count towards the final load time.  This is sometimes done to "prime" the cache (which by default is empty):

```javascript
// Automate the browser to request the home page.  This will prime the cache but will not contribute to the transaction load time.
driver.get("http://home.neustar");

// Now define the transaction.  Everything in here will contribute to the transaction load time.
test.beginTransaction(function() {
  // Now we're monitoring with a primed cache.
  driver.get('http://home.neustar');
});
```

## Define Your Steps
Transactions consist of logical steps.  For example, a *login* transaction has multiple steps:

- Go to homepage.
- Click on *sign in* button in the top right-hand corner.
- Enter username and password in the sign in form and click on the *login* button.

Defining your transaction as I've done above is a great first step when developing a script.  Now we just need to translate it into actions to perform and logically group those actions together with calls to the *beginStep* method:

```javascript
test.beginStep(description, timeout(ms), function(){});
```

Translating our 3 steps above into code:

```javascript
test.beginStep("Go to homepage", 5000, function() {
  driver.get("http://www.example.com");
});

test.beginStep("Click on 'sign in'.", 5000, function() {
  driver.findElement(By.xpath("//a[@id='sign-in']")).click();
});

test.beginStep("Perform login.", 10000, function() {
  driver.findElement(By.xpath("//input[@id='username']")).sendKeys("myusername");
  driver.findElement(By.xpath("//input[@id='password']")).sendKeys("mypassword");
  driver.findElement(By.xpath("//button[@id='login-submit']")).click();
});
```

There's a lot there.  Just focus on how we're using *beginStep*.  We can use the begin/end pattern just like we did with *beginTransaction*:

```javascript
test.beginStep("Go to homepage", 5000);
// Our code goes here
test.endStep();
```

What you put in a step is up to you and we could have squeezed all three steps into a single *beginStep* method call but that would reduce our visibility into the performance data collected.  You should generally have one call to *beginStep* for each page request made.

## Conclusion
Let's put everything together.  The *Test* interface allows you to create a client that will automate actions, define a transaction, manipulate the WPM proxy, and segment performance into separate steps for reporting purposes.  Something that makes our livese easier is that the *Test* interface is rolled into the global namespace of the script so we don't always have to be typing "test.":

```javascript
// Get the client we'll use for automating tasks.
var driver = openBrowser();

// I dunno...we may want to manipulate the proxy at some point :).
var client = driver.getHttpClient();

// Define our transaction and steps.
test.beginTransaction(function() { 
  test.beginStep("Go to homepage", 5000, function() {
    driver.get("http://www.example.com");
  });

  test.beginStep("Click on 'sign in'.", 5000, function() {
    driver.findElement(By.xpath("//a[@id='sign-in']")).click();
  });

  test.beginStep("Perform login.", 10000, function() {
    driver.findElement(By.xpath("//input[@id='username']")).sendKeys("myusername");
    driver.findElement(By.xpath("//input[@id='password']")).sendKeys("mypassword");
    driver.findElement(By.xpath("//button[@id='login-submit']")).click();
  });
});
```
