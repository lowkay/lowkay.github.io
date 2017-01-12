---
title: Help! My Awesome Webpack Configuration Isn't Building Incrementally!
author: lowkay
layout: post
---

This is a story about how a fairly complex Webpack configuration failed horribly and started building far more modules than it should do for incremental builds.

### TL;DR

There are a few reasons a module may be rebuilt unnecessarily:

1. You changed a file accidentally (even if you deleted the change and saved)
2. The name/path of the file being imported does not match the name/path of the file on the file system (case sensitive!)
3. The loader you use has a bug :O

A module being rebuilt affects all of the modules that depend upon it transitively.

It ultimately comes down to whether the Webpack cache contains a dependency and whether that dependency was last modified after the last build time.

You can find out by running `webpack-dev-server` with debugging using:

`node --inspect node_modules/webpack-dev-server/bin/webpack-dev-server.js`

This will give you a URL to throw into Chrome like this:

    Debugger listening on port 9229.
    Warning: This is an experimental feature and could change at any time.
    To start debugging, open the following URL in Chrome:
    chrome-devtools://devtools/remote/serve_file/@60cd6e859b9f557d2312f5bf532f6aec5f284980/inspector.html?experiments=true&v8only=true&ws=127.0.0.1:9229/ec025d22-52f9-48eb-af14-a063b88e5f12

Then navigate to `NormalModule` and breakpoint with a condition `!ts` [here](https://github.com/webpack/webpack/blob/8e69a808476aec661208437299edafd8554a5c12/lib/NormalModule.js#L305).

When you hit this breakpoint it signals that the currently executing module is going to be rebuilt because the current `file` has been modified or could not be found. Check that the `file` is normalized (uses forward slashes) and is correctly cased to the import in the executing module. You can find out the executing module and therefore which file to look at for the matching import by inspecting the locals and the value of `this`.

Resolve the issue either by fixing the normalization (which may only be possible by fixing the loader) or the path. If the breakpoint isn't hit or the names and paths are all good then something else is not working as it should - sorry, this post won't help :(

<!--more-->

### The story

At MindLink we have a fairly substantial collection of typescript projects and we moved to leveraging Webpack to do our bidding. Our setup is along the lines of:

- 5 independent typescript projects
- 3 of those are shared libraries
- 2 of them are applications that use those libraries

So we have Webpack configurations for the library tests and Webpack configurations for the applications.

We used Webpack 1 (as it was still the stable release) and [awesome-typescript-loader](https://github.com/s-panferov/awesome-typescript-loader) to get some decent incremental build performance.

We try to keep up to date with the latest tools so we recently moved to TypeScript 2 and that meant using [ts-loader](https://github.com/TypeStrong/ts-loader) (which does not support incrementally building the typescript files at the time of writing) because awesome-typescript-loader only supports TypeScript 2 with Webpack 2.

> sidenote: awesome-typescript-loader and ts-loader are merging their efforts, so in all likelihood we will be left with a single loader that will be the best of both!

Build times with this setup became a major pain point, we are talking *1-1.5 minutes* _for each change_! This pain needed sorting and we knew awesome-typescript-loader would help us out, but that we would need to move to Webpack 2 in order to leverage that. So we proceeded to migrate to Webpack 2 (the pains of which I should probably discuss in another post).

Once we migrated to Webpack 2 and awesome-typescript-loader was back in action we expected our incremental build times to become sane again (maybe several seconds). So we were **dismayed** when we found that a single change that should cause a rebuild of maybe 5 modules caused a rebuild of over _1000_! Sad...

Scouring GitHub issues and searching Google for similar experiences revealed that some people were having the same issues, but either they were solved by avoiding the issue, caused by a different issue or left unsolved. Now, of course, you'd find our solution that's actually been merged into awesome-typescript-loader!

What I decided to do was to *Look at the code!* Because when there isn't an answer already out there, you have to find it yourself.

Digging around the Webpack source I came across the cache and the mechanism that determines when something needs recompiling. The interesting bit is in `NormalModule` right [here](https://github.com/webpack/webpack/blob/8e69a808476aec661208437299edafd8554a5c12/lib/NormalModule.js#L305). Basically whenever a change is encountered each module is asked whether it needs recompiling and it decides this based on whether it or any of its dependencies were last modified after the last build time. If it cannot find the dependency it assumes it has never been built and if it finds one that is newer then it needs to be rebuilt. Both situations will trigger a rebuild of that module, but the semantics are important in this issue.

To find out what is causing modules to rebuild I followed these steps:

1. Launch webpack-dev-server using node inspection:
    `node --inspect node_modules/webpack-dev-server/bin/webpack-dev-server.js`
2. Open chrome dev-tools using the link that node spits out
3. Let it build the full application first
4. Set a breakpoint in `NormalModule` where it decides if it needs a rebuild
5. Make a change to a module that shouldn't impact many other modules
6. Wait for the breakpoint to be hit for a module that shouldn't be marked as needing a rebuild
7. Investigate why it needs a rebuild

Now I did this a few times and noticed something very strange - modules were being rebuilt because one or more dependencies were not found in the file system last modified collection. This is particularly intriguing because the files are obviously in the file system, I could see them, it built fine.  Then I noticed the discrepancy - the file system collection is indexed by a normalized file name i.e. forward slash separators, but the module paths were not normalized, they had backward-slash separators! So trying to find the module path in the file system collection always returned `undefined` and thus treated as though it's never been built.

Why this is the case is harder to determine, but if you ever see this it's likely a problem with the loader. Webpack expects all paths to be normalized for comparisons, so how did an un-normalized path get in? It isn't specified in our source code with backslashes.

I had a quick dig around how Webpack populates the dependencies of a module and it's all down to the loader.

The problem arises because the TypeScript compiler on Windows does not normalize the paths (well I mean it isn't its fault since why should it?) and awesome-typescript-loader was not normalizing the paths when passing the dependencies onto Webpack.

Additionally awesome-typescript-loader adds all of the TypeScript files in a project to the dependencies of each TypeScript module and that means if even one module is incorrect then all modules will be rebuilt.

The solution was to patch [this line](https://github.com/s-panferov/awesome-typescript-loader/pull/340/commits/163e53b542ac5cc3f579d7db3d286482a1b1e76e) - basically normalize the path being passed to Webpack.

However, after patching this issue we still noted that far more modules were being rebuilt - not the 1000+ any more, but several hundred - still not great!

Fortunately we now know where to look and running webpack-dev-server again under the inspector another issue was highlighted. Two issues exhibiting the same problems!

Whenever the import statement in the TypeScript doesn't match *exactly* the case of the file path there is another cache miss, because path normalization does not modify case and the cache is case sensitive. To fix these it is a case of modifying either the file name to match the import casing or the other way around.

Once all the casing issues were resolved it was suddenly all working as expected! Phew!

We went from recompiling over 1000 modules (and taking 50-60 seconds) to recompiling only those that really changed, often resulting in an incremental build of ~5 seconds. This drastically improved the development workflow and stopped wasting a lot of developer time waiting for a build to complete to see a change.

I would like to reiterate that if you're finding incremental builds aren't being very incremental then spend a bit of time investigating why and save your whole team lots of time :)