---
layout: post
title: Tools Section - HTML to ASP converter
date: 2017-05-03 00:00:00 +0000
tags:
- code
- tools
categories: []
---


So in my day to day work, I've been involved in a large scale conversion of an old ASP Classic web application to a .NET application. Nothing fancy, just a basic conversion to the new platform. Over 400 .asp pages in total, plus a lot of background and support work. No small task.

One challenge I ran into was taking sections of code that used if/then/else in-line with raw HTML. In some cases, I could keep this logic flow, much as I'm not a fan, and leave it alone. In some cases however, this logic needed to move to the code behind. This means the HTML needed to be put into response.write() calls.

This got old, very quickly. So I looked around for assistance, and found a [tool online, ](http://www.chrishardy.co.uk/asp/tools/html-to-response-write-converter.asp)designed for ASP Classic, however the syntax wasn't quite right.

So, I made my own. I decided to do it all in JavaScript, so that a page load wasn't required, and hosted it here on S3 as a static page.

If you're interested, [check it out here](https://tools.misaunders.com/responsewrite.html). I've actually set up a [base Tools page](https://tools.misaunders.com/responsewrite.html) which will list/index them all as I add others over time, but I expect I will make a new post here each time I add a new tool.