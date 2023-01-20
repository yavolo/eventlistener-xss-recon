There are many posts on how to use eventlistener for XSS already, but not so many on how to find them with recon and tools.

I suggest reading writeups on eventlistener XSS before getting into this, since im skipping the technical parts to focus on methodology.

<h1>Recon</h1>
Initially, I look for this regex in all the included js with this regex:

`cat hosts | getJS | grep target.com |  httpx --match-regex "(?i)addEventListener\((?:'|\")message(?:'|\")"`

`grep target.com` will narrow the results to not include public static CDN servers. These are maintained and hard to exploit.
However if the target have their own private content delivery on e.g. static.target.com it may not be as well maintained and hold
vulnerable thirdparty js files. This is mainly what I look for when checking these results.

And of course attached eventlisteners can also exist in script tags on normal pages:

`cat hosts | hakrawler -plain |  httpx --match-regex "(?i)addEventListener\((?:'|\")message(?:'|\")"`

<h1>Method</h1>
With these results, we can start checking the pages. I mainly use https://github.com/fransr/postMessage-tracker on chrome to sift through pages. It tells where the scripts come from in the top corner and prints all transfering messages in the console. 

Burp also comes with a similar function in their embedded browser(Dom Invader).

If a message is sent that looks "home made" or occurs during a sensitive flow in the application, I take a look at it.

<h1>Exploitation</h1>
Tomnomnom brings up some valuable tips on how to work on postmessage XSS on STÖKs channel: https://www.youtube.com/watch?v=FTeE3OrTNoA

Some key take aways is how valuable the chrome debugger is, open your console -> go to the sources tab -> Global Listeners -> message. That's all the registered
message eventlisteners on the page. 

Put a breakpoint at the listener and shoot `window.postMessage('test', '*')` in your console and see where it goes from the breakpoint. The fun sport is to try to please the if()'s and regexes until it hits a sink or similar.

If the window.postMessage() pops from your console, and the origin isn't properly checked(The boring part that prevents exploitability many times), make a post on
https://repl.it like Tomnomnom suggests, with the following template code:

```
<!DOCTYPE html>
<html> 
 <head>
   <script>
var target = document.getElementById('target')

target.addEventListener('load', () => {

target.contentWindow.postMessage({
    "type": "redacted",
    "data": "<script>alert(document.domain)</script>"}, '*')
})
target.src = "https://test.target.com/search?q=yavolo"
   </script>
 <meta charset="utf-8">    
 <meta name="viewport" content="width=device-width">   
  </head>
    <body>  
       <iframe id=target></iframe>  
    </body>
</html>
```

If you see the target domain and not repl.it in the alert prompt, its bounty time. Just paste your `repl.it` link in your Hackerone report.



<h1>Practice</h1>
Try it out here:

https://public-firing-range.appspot.com/dom/toxicdom/postMessage/eval

https://public-firing-range.appspot.com/dom/toxicdom/postMessage/innerHtml

https://public-firing-range.appspot.com/dom/toxicdom/postMessage/documentWrite

https://public-firing-range.appspot.com/dom/toxicdom/postMessage/complexMessageDocumentWriteEval

https://public-firing-range.appspot.com/dom/toxicdom/postMessage/improperOriginValidationWithPartialStringComparison

https://public-firing-range.appspot.com/dom/toxicdom/postMessage/improperOriginValidationWithRegExp


Twitter:
@oliverrickfors
