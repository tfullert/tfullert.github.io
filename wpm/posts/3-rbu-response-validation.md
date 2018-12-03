## Introduction
The last three posts have focused on scripting compontents that serve to _automate_ the browser's activity.  Today we'll focus on the basics for _validating_ the response that the browser receives.  In particular we're going to look at two main approaches for validating a response:

- Verify load time of the response.
- Verify content within the response body.

There are numerous ways to do both, today we'll go over a few basic examples for each.  We'll certainly revisit more advanced verification techniques in future posts.  The _beginStep_, _waitForNetworkTrafficToStop_, and _waitFor_ methods are all part of the [Test Interface](http://docs.wpm.neustar.biz/testscript-api/biz/neustar/wpm/api/Test.html).

## Load Time Verification
The time it takes for a page or asset to load is one of the key reasons we perform external monitoring.  Customers will not stick around for long if each page takes 20 seconds to load (could you imagine!?!).  I wont dwell on the [impact slow load times have on revenue](https://medium.com/@vikigreen/impact-of-slow-page-load-time-on-website-performance-40d5c9ce568a), but instead will show you the quick and easy way to alert yourself to slow loading pages with WPM.  There are three ways.  First, using the timeout parameter to the _beginStep_ method:

```javascript
// Alot 30 seconds (30000 milliseconds) for the step timeout
beginStep("Go to homepage", 30000, function() {
  driver.get("http://home.wpm.neustar.biz");
});
```
If the actions within the _beginStep_ take longer than 30 seconds to perform an error will be thrown.  This is the easiest way to measure page performance but also the least flexible.  Another approach you can take is to _waitForNetworkTrafficToStop_:

```javascript
beginStep("Go to homepage", 30000, function() {
  driver.get("http://home.wpm.neustar.biz");
  
  // Look for a 2 second pause in traffic and throw an error if one not found after 30 seconds 
  waitForNetworkTrafficToStop(2000, 30000);
});
```

I've decided to keep the original step timeout value in there (the second parameter to _beginStep_) but the real check comes with _waitForNetworkTrafficToStop_ which starts a 30 second time and also sets a condition that a 2 second pause in traffic should be observed during that 30 second timer.  The moment a 2 second pause in traffic occurs the page load is confirmed and the script will move on to the next step.  One problem with this approach is that if your page is continuously receiving a feed of asynchronous content (ex: triggered by client-side JavaScript) then you may never see a 2 second pause in traffic.  So, this approach is typically useful for static or monolithic pages.  Anything that is dynamic or has a continuous feed of data will almost always throw an error when using _waitForNetworkTrafficToStop_.  Which brings us to the third approach, _waitFor_:

```javascript
beginStep("Go to homepage", 30000, function() {
  driver.get("http://home.wpm.neustar.biz");
  
  // Wait for a login button to be present on the page
  waitFor(function() {
    return driver.getElement("//button[@type='submit']").getText().toLowerCase().startsWith("login")
  }, 20000);});
```

This approach uses the _waitFor_ function which takes two parameters; the first one is a callback function which must return true or false, the second parameter is a timeout value in milliseconds.  The _waitFor_ function waits for the user defined (callback) function to return true and will wait up until the timeout has expired (ex: 20 seconds).  This is a much more flexible check since you can perform multiple checks within the callback function and then consolidate the results into a single true/false return value.  

## Content Verification
While it's nice to get a quick response from a web server for a page we request, we definitely want to verify that we've received the "correct" response.  For example, if I enter my username & password into a login form and click on submit but I only get a blank page in response then that is not good.  What about when I perform a login and I get a splash page saying that the site is currently unavailable?  If a web server experiences a 500 Internal Server Error it does not (or I should say, should not) return the 500 ISE and stack-trace.  Instead the web server should be configured to return a 200 OK response with a maintenance splash page.  If you're not performing an extra content validation in your script then such a response will look like succes. Not good!  The basic content validation I do is to search the text of the HTML body tag of the page.  This is the quickest approach to content validation and is done as follows:

```javascript
beginTransaction(function() { 
  beginStep("Go to homepage", 30000, function() {
    // Make a page request
    driver.get("https://home.wpm.neustar.biz/");
    
    // Check page load time first
    waitForNetworkTrafficToStop(2000, 15000);
    
    // Check the content of the page body
    var bodyText = driver.findElement(By.tagName("body")).getText();
    if (!bodyText.contains("Create an Account")) {
      // thrown exceptions will also cause the monitor to record an error
      fail("Expected content not found!");
    } else {
      log("Found: " + content_check);
    }
  });
});
```

We want to perform the page load time check _before_ the content check because we want to make sure that the content should be there before we check for it.  The content verification occurs on line 23 and is limited in scope to just the innerText of the HTML body tag.  Because we've limited the scope of our search to just the body tag you will want to check in a browser to make sure the text you are testing is indeed in the HTML body.  You can do this using the browser developer tools that come with any of the major browser versions (I prefer Google's DevTools).  The developer tools allow you to inspect various elements of the page, for example I can execute _document.body.innerText_ from the console and it will show me all the text within the body tag of the page.  I should be able to pick any sub-string within the text returned and use it for my search string.  If you're using Google Chrome you can find out more about Chromes Tools for Web Developers [here](https://developers.google.com/web/tools/chrome-devtools/console/expressions).
