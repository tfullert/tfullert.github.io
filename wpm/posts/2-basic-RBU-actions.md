## Introduction
Last time we discussed some [basics of the *Test* interface](posts/1-test-interface.md) and how it can be used to segment automation actions (re: HTTP page requests) into logical steps for the purpose of script development and eventual graphing/reporting.  That code provides the bones of our final script and as a recap looks like this:

```javascript
// Get the client we'll use for automating tasks.
var driver = openBrowser();

// I dunno...we may want to manipulate the proxy at some point :).
var client = driver.getHttpClient();

// Define our transaction and steps.
beginTransaction(function() { 
  beginStep("Go to homepage", 5000, function() {
    // Code to request the homepage.
  });

  beginStep("Click on 'sign in'.", 5000, function() {
    // Code to click on the sign in button in the top right-hand corner.
  });

  beginStep("Perform login.", 10000, function() {
    // Code to enter our username & password into the login form and then click on submit.
  });
});
```

Today we're going to add the meat to those bones by adding the basic WebDriver code that will automate actions within our browser.  There's a lot of different things you can do with WebDriver but there are really only 3 things you'll be doing most of the time (*60% of the time it works every time*):

- I want to go to something
- I want to click on something
- I want to type something

So we'll be focusing on the WebDriver code associated with those three actions today.

## The Trinity
Remember the personal computer [Trinity](http://www.historyofpersonalcomputing.com/category/trinity/) of the late 70's?  Well there's a WebDriver Trinity as well.  These are the 3 main WebDriver commands that you're going to use most of the time:

```javascript
// I want to go to something
driver.get(URL);

// I want to click on something
driver.findElement(By by).click();

// I want to type something
driver.findElement(By by).sendKeys();
```

Woah!  What is *By by*? In WebDriver, for situations where you have to interact with an HTML element you first need to find that element.  You do that using the *findElement* method which takes a Locator as a parameter.  There are a number of different strategies that can be used for locating an element but my favorite is by xpath so I'm going to use that here.  We'll do a separate blog post on locator strategies some day but for today just be aware that we're using a query language called xpath that allows us to select HTML elements from a web pages Document Object Model (DOM) by writing crazy stuff like this:

```javascript
driver.findElement(By.xpath("//a[@id='login']")).click();
```

Our xpath expression is "//a[@id='login']" which looks for an anchor tag (anywhere in the DOM) with an id attribute of 'login'.  The end result is that *findElement* returns a [WebElement Object](http://toolsqa.com/selenium-webdriver/webelement-commands/) which we will then call the *click* method on.  We're saying, "hey browser, find an anchor element with an id attribute of 'login' and then click on it".  If I wanted to perform a click on a button that doesn't have an id attribute but instead has a class value of 'loginBtn' then I could write the following code:

```javascript
driver.findElement(By.xpath("//button[@class='loginBtn']")).click();
```

So easy!  Remember, you can use browser extensions such as Chrome's [DevTools](https://developers.google.com/web/tools/chrome-devtools/) to determine an objects xpath expression.

## Sample Code
Let's extend our code from the last blog post.  I'll use the [WPM portal login](https://home.wpm.neustar.biz/) page as an example.  To make things a bit easier I'm going to forget about going to Neustar's homepage and will just start directly on the WPM login page:

- Go to login page.
- Enter username and password in the sign in form and click on the login button.

The first step is very easy because all we have to do is provide the starting URL we want to visit:

```javascript
beginStep("Go to login page.", 5000, function() {
  driver.get("https://home.wpm.neustar.biz/");
});
```

The way the step is defined we're going to wait 5 seconds for the page to load.  We should check the resulting page for specific content and we will during a future blog post.  But for now we're not adding any additional checks beyond what we get from the step definition.  The result of this step is that the browser will load the index page of home.wpm.neustar.biz.  

That page will contain a login form and that form will contain two text fields (one for username and one for password) as well as a button to submit the form (i.e. login).  In our next step we want to find the two text fields, enter our username & password into those fields, and then click on the button to submit the form.  This all happens within a single step definition.  Using DevTool's inspection feature (mouse over the element, right-click, select inspect) we can generate xpath expressions for the two text fields as well as the login button:

```javascript
beginStep("Perform login.", 10000, function() {
  driver.findElement(By.xpath("//input[@id='username']")).sendKeys("MY_USERNAME");
  driver.findElement(By.xpath("//input[@id='password']")).sendKeys("MY_PASSWORD");
  driver.findElement(By.xpath("//button[contains(., 'Login')]")).click();
});
```

When I went to construct an xpath expression for the login button I noticed that there weren't very good HTML attributes defined for the button.  There was no *id* (my go-to attribute), *name*, or *class* attributes, so I ended up using the text of the button.  Furthermore, the text contained a character (maybe extended ASCII) that I didn't really want to deal with so I just searched for the partial text "Login".  Sometimes you'll be writing an automation script for a web application that doesn't define lots of attributes and that can be tough.  You might want to try CSS selectors or you might have to base your xpath expression more off the structure of the page (ex: /html/body/table/tr[0]/td[0]/form/input[0]).  That second approach is nasty and can break easily so I'd recommend against it.

## Conclusion
Taking this all together with our original script framework we now have a working script that we can use to monitor the WPM login page:

```javascript
// Get the client we'll use for automating tasks.
var driver = openBrowser();

// I dunno...we may want to manipulate the proxy at some point :).
var client = driver.getHttpClient();

// Define our transaction and steps.
beginTransaction(function() { 
  beginStep("Go to login page.", 5000, function() {
    driver.get("https://home.wpm.neustar.biz/");
  });

  beginStep("Perform login.", 10000, function() {
    driver.findElement(By.xpath("//input[@id='username']")).sendKeys("MY_USERNAME");
    driver.findElement(By.xpath("//input[@id='password']")).sendKeys("MY_PASSWORD");
    driver.findElement(By.xpath("//button[contains(., 'Login')]")).click();
  });
});
```

This code can also be used as a template for other web application login that have a similar workflow.  You would just need to change the starting URL and change the xpath selectors for the *sendKeys* and *click* methods.  There is a crucial part missing from our script and that is some sort of content validation.  And that will be addressed another day :).
