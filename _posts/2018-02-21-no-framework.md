---
title: Why no framework can be the best framework (opinion)
author: lowkay
layout: post
---

The JavaScript ecosystem is fragmented. That’s not a bad thing. It does make life difficult when trying to decide what framework(s) to leverage in new and existing projects. You don’t want to be locked into a framework that becomes stale while your project is actively developed for a variety of reasons (I plan on another post to describe situations when you might not want to care).

There was a sizable movement towards micro-frameworks to alleviate some of the issues with lock-in, but also to make things more focused and maintainable. Of course that also has its downsides. However, there still remains many more complete do-it-all frameworks or popular combinations of frameworks. Some frameworks appear to be modular, but in reality you're probably going to end up using all the modules together anyway.

The problem is when you have a long-running project you need a framework that is stable and will remain maintained and stable as you continue to use it. You may worry about choosing a certain combination of frameworks because you can’t be sure they will continue to exist and aren’t just some current fad in the fast moving world of npm packages.

Or you could think about what is critical project code and what is not. Then avoid leaning on frameworks for your core critical code.

# TL;DR

Use an appropriate framework, but keep your core business logic out of a framework (or use a simple in-house framework).

<!--more-->

# The MindLink Frameworks Journey

From my experience at MindLink I have had to deal with very large legacy code bases and long-running projects that have surpassed the usefulness of the chosen framework. The main reason is maintainability. Everything in the web ecosystem is moving faster and faster, frameworks that do too much stop being useful and start getting in the way. It isn’t their fault, they’re built to deal with the web and take away pain, they abstract things and unify differences in browsers or devices. That sounds like a win. It very much depends on the framework.

The crucial thing is making sure that at the very least there is a feasible migration path from legacy code leveraging one framework to a newer framework (or newer version of the same framework). When there isn’t you are faced with a few choices:

1. Start hacking around the framework leading to spaghetti code and lots of time spent fixing the framework
2. Start fresh - rebuild everything and take on the challenge of keeping (or improving) the current feature set
3. Leave it with its quirks and keep building on top of it, probably leading to instability and a lack of ability to leverage new web features
4. Maintain the framework yourself (if you can) and take on the burden of keeping it up to date

None of those are particularly appealing. At MindLink in the past we have taken on at least 3 of those approaches.

In 2006 we started out with a code base that was essentially an untested (somewhat untestable) intern project that grew and grew into a partially structured beast. At that time our web app was built with IE6 in mind and back then it was almost impossible to have a cross-browser site that worked without using a framework - we leveraged [ExtJS](https://www.sencha.com/products/extjs/#overview) (1 and then 2 - it's moved on a lot since then).

The problems we faced as time went on included, but were not limited to:

1. Bugs in ExtJS layout code on newer browsers requiring monkey-patching
2. Lack of being able to take advantage of new browser features due to the lock-in of ExtJS’ layout system
3. Lack of being able to take advantage of new browser features due to the ExtJS’ class system and loader
4. Inability to continue upgrading Ext versions due to breaking changes in major versions

This pain was felt for some time. When we were tasked with overhauling our web UX in 2011 we didn’t have time to rethink the framework. We had locked ourselves into the ExtJS ecosystem. It wasn’t feasible in the time we had to migrate away from Ext.

As a result we stuck with ExtJS, albeit with a newer version (ExtJS 4). We took the core business logic (still in Ext’s class system) and built a compatibility shim module to monkey-patch the new ExtJS to expose APIs and behaviours that the old code expected. The win there was that we only had to code the new UX from scratch, and even then we found we could reuse some bits that weren’t too ingrained in the old API. However, we still had to live with a layout system, class system and loader that were proprietary and thus not compatible with new trends. It was painful to implement the new UX and there was a lot of working around the framework, leading to new features taking longer than expected to implement.

Fortunately at that time we turned our attention to mobile just as hybrid applications were becoming viable (circa 2013). The ecosystems for multi-platform mobile web applications were limited. We knew one thing though, we didn’t want to tie our core business logic to any framework and lose the portability when we didn't **need** the framework.

We still needed something to take the UX pain away for mobile device discrepancies. Fortunately our old friend Ext had a relative called Sencha Touch (since merged with the new Ext). Unfortunately it still carried with it a class system, loader, software patterns (MVC), utilities and fluff. Being used to Ext it wasn’t a huge leap and there wasn’t much in the way of choice at the time.

What we did realise early on is that we didn’t really like the fit of the patterns that worked by default with Sencha Touch. A single controller for multiple views made it harder than it needed to be when dealing with different instances of the same view and for our product that would be a common occurrence. We set about with our new framework-free core and built a view-model layer upon it. That view model layer made it easier for our developers to reason about, but it came at a cost. By not using the framework’s go-to patterns we ended up fighting it and giving ourselves more boiler-plate code to tie the two together. That’s pain we still live with. The framework had an affinity to a certain way of doing things and while it was 'flexible' that didn’t mean it wasn’t painful to deviate - some frameworks today still have their own special way.

When the time came to rejuvenate our web app again (2015) the JavaScript ecosystem had changed. React and the virtual DOM were breaking onto the scene and challenging the likes of angular with a micro-framework approach. What makes React powerful in my opinion is that it does just the one thing - the view - and isn’t opinionated about how it’s driven. The choices we made when building our mobile client paid dividends here - with all of our business logic in our own framework we really only needed the view. This is also where our view model choice paid off, because we could leverage the same view models in mobile and web applications despite the views often being different.

User interfaces are not always simple, and at MindLink they are fairly complex - rich text editing, complex resizable layouts, dialogs and others. So while we “only” had to deal with the view that’s still a lot of work. By leveraging React and it’s un-opinionated style we created base React components to bind a component to a view model and leveraged that for most of the components, thus reducing boiler-plate.

We quickly found productivity was much improved when working with React and our own core compared to working with Sencha Touch and we looked to find a way to migrate away from Sencha Touch (by this time outmoded and outdated) and leverage React. This was possible through replacing views in the mobile client with React portals and connecting the lifecycle of a React component with the Sencha view that contained it. This increased code reuse further and reduced boiler plate. However, it’s a long process because Sencha views work with Sencha stores and Sencha classes and the Sencha loader (the do-it-all framework curse). The framework that started us off with a jog, gradually slowed us to a walk and nearly a crawl. Simply, it did too much. I mean it needed to at the time because there was no ECMAScript 2015 with its modules and classes and there weren’t sophisticated bundlers around. Had we went all-in with Sencha we would not have been able to build half of what we have in the same time frame.

So there’s the clincher - use a framework, it will undoubtedly take pain away, but don’t use it for your core business logic. You need your core logic to be portable, reusable, timeless and in your control. Functional programming encourages this - you don’t need frameworks to write functions and hook them together, I think that’s part of the reason why they’re becoming more popular.

I don’t want to take anything away from frameworks big and small. They work and when applied to the right problems they work very well. But don’t think you *have* to use a framework to be productive, they’re tools that can be helpful in the right context.

Now at MindLink we have a core framework-free TypeScript code-base that contains our client-side business logic and common view models. That accounts for at least half of our mobile + desktop code base. We get a little more reuse from a common React layer (perhaps 10%) and we are still working on ways to further increase code reuse across our client stack.