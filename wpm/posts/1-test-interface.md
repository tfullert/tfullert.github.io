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
var client = driver.openHttpClient();
```

This will allow you to intercept/rewrite headers and perform other tasks that can benefit your RBU test.  In the end, the goal is to get a client that you can automate for performance monitoring.  A good way to determine which client type to instantiate is:

- HttpClient if: 
-- You want to do basic tests of GET/POST performance & availability.
-- You want to test an API or other endpoint that only returns text.
-- You just want to measure availability of a page.
- WebDriver + HttpClient if:
-- You want to monitor complex applications that rely on client-side code.
-- You want to simplify script development.
-- You want to have control over the WPM proxy that will make the requests.

For real browser scripts it almost always makes sense to grab instances of both the webDriver and the HttpClient.

## Establish A Transaction

## Define Your Steps

## A Few Other Things

## Conclusion
