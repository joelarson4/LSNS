LSNS: A localStorage key namespacing convention
===============================================

This repository has a single purpose: to iron out a convention for HTML5 `localStorage` key namespacing.  All contributors are welcome to either submit problems with the current proposed rules as issues, or to fork and request-for-pull any changes.

This idea began its public life here: http://joewlarson.com/blog/2012/12/02/we-need-a-standard-for-namespacing-localstorage-keys/

The Problem
-----------
There is a trap hidden within the promise of widespread use of HTML5′s localStorage feature. The trap is the fact that the `localStorage` for a particular subdomain is shared by all scripts that run on that domain. This includes both application code and library code.

If multiple scripts running on a subdomain uses `localStorage`, it is easy to imagine them conflicting with each other. For example, if two or more scripts use some of the same `localStorage` item keys, they will trample on each other’s data, possibly causing each other to choke on unexpected data or to be fooled by plausible but incorrect data set in place by another script. Or, if one of the scripts uses `localStorage.clear()` when it decides its cache has gone stale, it will clear out every other script’s data as well. This might cause unnecessary repeated downloads of data that was in `localStorage` only moments before.

It might be reasonable to expect that all of the application code (by which I mean non-library code) on a single subdomain should be coordinated in usage of `localStorage` (though if multiple teams are involved even this might be a stretch). However it’s definitely not reasonable to expect that different libraries will be so coordinated. At least not right now.

It would have been nice if `localStorage` had had namespacing built into its API. Perhaps the `localStorage` API can be expanded to include this in the future.

In the meantime, another approach is to introduce a library which provides namespacing for `localStorage`. Squirrel.js is one such library and appears to be well thought out (though I haven’t used it). However, I doubt that every library author would want to add this to their list of required libraries, just as web designers probably don’t want one more script they have to include in all their projects.

The Solution
------------
LSNS will be a "convention" (not a standard -- that'd be too heavy), a set of rules which JavaScript libraries can employ to use `localStorage` in a fashion that won't collide with other `localStorage` usage.  The rules must meet these requirements as far as keys:

 - They must not be excessively long, because key length counts against you in the total amount of `localStorage` your domain can use.
 - They should not include *https://* or *http://*, primarily because this adds to key length and provides no really useful information.
 - They should be fairly predictable -- it should be possible to guess what a script's key is or figure out what script a particular key goes with.
 - They should require no central authority to acquire -- it should be possible to just create one that follows the rules.

With those requirements in mind, the rules for LSNS will be:

 - For application code, `localStorage` keys should be prefixed with the root subpath within the website that represents the application, followed by a colon. For example, "/stocks:", "/admin:", or just ":" if this is code for the whole site.
 - For libraries, `localStorage` keys should be prefixed with the subdomain of the script’s primary "home" on the web, followed by a colon. For example, "github.com/theAuthor/theScript.js:" or "scriptName.somesite.com:".
 - All libraries should supply a documented means for using a different namespace. For example, `SomeLibrary.setLsNamespace(…)` or `new SomeLibrary({lsns: ‘…’})`.
 - Libraries should avoid using `localStorage.clear()`.
 - Libraries should note prominently that they "implement the LSNS convention using a prefix of ____".
