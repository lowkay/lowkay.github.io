---
title: Exchange 2013 - not a bitwise exclusion
author: lowkay
layout: post
---

A few months ago I was implementing a bug fix involving Exchange integration. Our product supports Exchange 2010SP2 and above, so modern(ish). We leverage Exchange Web Services (EWS) to perform operations on a user's mailbox. One such operation, and related to the bug being fixed, requires a search for items that have a specific flag set.

The `MessageFlags` field is the most common of these, so if I want only read messages I would perform a search that restricts results to items that have the *read* bit set.

Unfortunately EWS only supports an exclusion bitflags operation which is documented as *"a bitwise AND that will match only if the result is 0"*. So if we want to filter by read (i.e. The bit is set) we cannot use exclusion alone.

Fortunately EWS also supports *NOT*, which matches items if and only if the inner expression did not match.

So a query that looks like this `NOT(EXCLUDES(READ))` should suffice.

And it does, for the most part... applying the same logic we saw strange results - notably Exchange 2010 and 2016 were working as expected, but Exchange 2013 was not doing what we expected. In fact it was doing the opposite!

# TL;DR

Exchange 2013 => not a bitwise exclusion:

- 2010 passes
- 2013 fails
- 2016 passes

Cause - 2013 is "optimising" the query and performing a negated bitwise AND rather than performing the bitwise AND and then negating the result.

<!--more-->

# The Journey

Initially I felt that it most likely is our fault, after all would MS **really** break EWS in 2013 and nobody has experienced it?! After spending an hour debugging, checking the requests to Exchange, making sure the items are actually where they should be etc. I decided to revisit the assumption that Exchange was fine.

Now the `not..excludes` query is awkward, why is there no inclusion bitwise query? Well, because with NOT and EXCLUDES you can get the same semantics, true. The fact that we see inverted results suggests that actually what's happening in Exchange is that the query is being treated as inverted, as if the NOT is ignored... or... as if the NOT is applied first!

Time to test the hypothesis, if I bitwise negate the READ flag test of `0001` I get `1110` and if we pass that as our EXCLUDES filter, in theory that will be flipped back and then applied to the search results... after that small tweak we strengthen our hypothesis - Exchange 2013 now works, but 2010 and 2016 fail :(that makes sense of course since `!(A & B) != !A & B` of course I cannot guarantee this result, it seems bizarre and likely very specific to an excludes expression wrapped in a not.

So that leaves us with a choice - determine which server we are talking to and construct the query accordingly, or perform the filtering client-side.

Going forward we may find that the semantics of the query change with future versions of Exchange too, they shouldn't, but how confident can we be given that they definitely have in the past? We also want to reduce code branching as that makes the whole thing more complex.

By doing the filtering client side at least we have control over the semantics, at the cost of superfluous items. Fortunately the cases where this filtering out takes effect are small and we are over a trusted server-server link so bandwidth isn't a concern (at least at this time).

As a result we implemented a client-side filter and fixed the bug quickly (considering our first attempt fell foul of an Exchange protocol change).