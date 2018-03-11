---
title: Why you need a framework, possibly (opinion)
author: lowkay
layout: post
---

The big caveat here is that the project has to be at least complex enough that static HTML or simple JavaScript won’t suffice.

If you are faced with a project that has a definite shelf life of under a couple of years then you probably want to use a framework. Otherwise you may want to think carefully about when and how to use a framework.

<!--more-->

Frameworks exist because developing everything from scratch is painful. There’s lots to think about and when you have a deadline for a project you want to focus on the business logic not on boiler plate or utility functions.

Frameworks are not always good. They have a definite learning curve (some steep, some less so), they introduce more dependencies, they may force you into structure or patterns and they may not work just how you’d like them to. More importantly the web moves fast and frameworks come and go, drop out of maintenance and you have little to no control over that.

It’s a trade off, but in my opinion a framework should be leveraged for projects where:

1. Its complex enough that vanilla JS or a bit of jquery (and I mean really not much at all) can’t do the job
2. The shelf life of the project is short - max a couple of years
3. There is time to learn a suitable framework OR a suitable framework is already familiar
4. You strongly agree with the approach a particular framework takes

Suitability is very important - you don’t need a hefty framework if you’re only going to be leveraging a fraction of its capabilities - it will only serve to slow the learning for other developers on the project.

It is also very important that a common structure and a known way to do things is instilled into a complex project. There’s multiple ways to do things and if the project has no framework that has best practices / structure then it will often end up with multiple different approaches. If there’s no framework in use those practices are internal and over time the choices that you make may change, you forget why you did one thing one way and another thing another way. New developers get confused with the right approach and that’s cognitive overhead you don’t need when you’re trying to deliver. With widely used frameworks best practices are common and you don’t have to spend time defining your own.

On top of the importance of structure is how much pain the framework takes away from you - depending on the project that could be things like synchronizing data with the UX or making web requests and processing the responses.

Ultimately in short-lived projects you don’t have to worry as much about a chosen framework going out of date - the project itself will be gone by then. When you don’t have to worry about maintaining the project beyond the framework lifetime you can allow yourself to lock-in to the way that particular framework is designed.