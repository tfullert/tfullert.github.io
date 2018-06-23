## Introduction
Neustar's Web Performance Monitoring (WPM) platform is an external performance monitoring and load testing tool for evaluating web application performance.  Central to that purpose is an automation script that tells the platform what actions and HTTP requests need to occur in order to evaluate performance.  For example, if I want to ensure the performance and availability of my web application's login process then I need to create a script that automates the following actions:

1. The user navigating to the login page.
2. The user entering their username & password.
3. The user clicking on the *login* button.

Script development is actually pretty easy.  There are only a few concepts and technologies that you need to be familiar with:

- Virtual Users & Real Browser Users
- A Browser's Web Development Toolkit
- JavaScript
- Selenium 3.0
- Apache HttpClient

Sure, there are other things you can (and should) be familiar with such as xpath expressions, the DOM/render trees, and HTML.  But if you want to get started on writing your own scripts then the five topics listed above should suffice.

## Virtual Users & Real Browser Users
The are two types of automations that we can perform; high-level and low-level.  

A high-level automation involves writing a script that controls a browser, this is referred to as a Real Browser User (RBU) on the WPM platform.  Neustar uses an open-source automation framework called Selenium for this purpose and with it you can write very simple code that automates clicking or typing text in a browser.  You get a lot of work for free.  For example, a click on a button within a form may call a bit of JavaScript that performs some action or manipulates the outgoing HTTP request.  With RBUs you don't have to worry about that in your coding, the browser's JavaScript engine handles the execution of the code.  Because of this RBUs are suitable for web applications that have a rich front-end with lots of dynamic behavior driven by JavaScript.    

A low-level automation involves writing a script that emulates the underlying HTTP calls made by a browser, the GET & POST requests that occur when you navigate a web application.  This is referred to as a Virtual User (VU).  VUs are similar to a wget or cURL request and therefore they are just making simple HTTP requets, they are not parsing/processing the HTML that is returned and they do not have a JavaScript engine that will execute the JavaScript.  Going back to the login process example given earlier, when you click on the login button ultimately what happens behind the scenes is that the browser bundles up the text in the form fields and submits them to the server using an HTTP POST request.  Just this simple process can be quite difficult because you have to account for all the content that needs to be added in the POST body.  This could include hidden values as well.  Also, if the or target URL undergo a transformation due to some JavaScript execution you now need to write code to do that.  

So which approach do you use?  That depends on what you're trying to do.  VUs are good for: testing APIs (which have text responses), uptime-only testing, high concurrency (>50K concurrent users), and the price conscious.  RBUs are good for: testing web applications with rich front-ends and for monitoring & load testing jobs where highly accurate representation of end-user performance is desired.

## A Browser's Web Development Toolkit
Regardless of whether your writing VU or RBU scripts you are going to need to be able to inspect a web application.  Most modern browsers have a toolkit built into them that allows you to analyze a web application being viewed in the browser.  For script development we'll use these browser toolkits to figure out what HTTP request is being made when you click on a button (for VUs), how to identify elements within a page by their xpath expression (RBUs), or identify what content should appear on the page for validation (both RBU & VU).  I personally prefer Chrome and its [DevTools](https://developers.google.com/web/tools/chrome-devtools/) so that's what I'll recommend here.  There are other methods for doing this as well (Wireshark, Fiddler, manual review of page source) but a browser toolkit will be the most effective at script development and preserving your sanity!  So read up on your toolkit's documentation and pay special attention to the following topics:

- [Console Tab](https://developers.google.com/web/tools/chrome-devtools/console/) - Great for [testing out xpath expressions](https://stackoverflow.com/questions/22571267/how-to-verify-an-xpath-expression-in-chrome-developers-tool-or-firefoxs-firebug).  I also use it to test some JavaScript that I'm having difficulty with.
- Inspecting an element - Just right-click on an element (link, button, image, etc.) within the web page and there should be an *inspect* context-menu item.  Select that and it will take you right to the element within the source/DOM.  It's that easy!
- [Network Tab](https://datawookie.netlify.com/blog/2016/09/view-post-data-using-chrome-developer-tools/) - Inspecting the body of a POST request is almost as easy as inspecting an element!

## JavaScript

## Selenium 3.0

## Apache HttpClient
