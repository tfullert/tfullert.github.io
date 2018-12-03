## Introduction
The last three posts have focused on scripting compontents that serve to _automate_ the browser's activity.  Today we'll focus on the basics for _validating_ the response that the browser receives.  In particular we're going to look at two main approaches for validating a response:

- Verify content within the response body.
- Verify load time of the response.

There are numerous ways to do both, today we'll go over a few basic examples for each.  We'll certainly revisit more advanced verification techniques in future posts.

## Content Verification
While it's nice to get a response from a web server for a page we request, we definitely want to verify that we've received the "correct" response.  For example, if I enter my username & password into a login form and click on submit but I only get a blank page in response then that is not good.  What about when I perform a login and I get a splash page saying that the site is currently unavailable?  If a web server experiences a 500 Internal Server Error it does not (or I should say, should not) return the 500 ISE and stack-trace.  Instead the web server should be configured to return a 200 OK response with a maintenance splash page.  If you're not performing an extra content validation in your script then such a response will look like succes. Not good!  The basic content validation I do is to search the text of the HTML body tag of the page.  This is the quickest approach to content validation and is done as follows:

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
