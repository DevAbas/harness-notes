# Scaffold — authentication

Task: see scaffold-auth-prompt.txt
Branch: scaffold/auth
Harness state: layers 1, 1b, 2 and 3 in place
Prompt style: detailed spec — scaffold work, not a measurement
Baseline: harness-01b
Codebase: ~7 screens, ~180 source files, 406 tests
Files touched: 23 modified, 13 new
Lint: 19 warnings, unchanged from the pre-change baseline

Scaffold work. Login and register, httpOnly session cookie, every existing
route protected. It replaces the role switcher on the Settings page, which
existed because the server had no authentication to enforce — the Settings
page itself went with it, on my call, since nothing else was left on it.

Recorded because one finding here is the most consequential single defect
in the whole record, and because two of the harness layers can be seen
working for the first time.

## auth-1 The session cache is written to an object nobody is watching
Signing in or registering ran `queryClient.clear()` and then
`setQueryData(sessionKeys.me(), session)`.

After `clear()`, those are two different Query objects under one key. `clear()`
destroys every Query and drops it from the map. A mounted `useSession` is
subscribed to the Query object, not to the key, and removal neither detaches it
nor tells it. The next line builds a brand-new Query and writes to that one,
with zero observers. The cache is correct and every mounted consumer is still
pointed at the destroyed instance.

An orphaned observer recovers on its next render. RoleProvider sits above
BrowserRouter, so the post-sign-in navigate() re-renders everything below it and
never the provider. Nothing re-renders it, so it holds the role it last rendered
until a reload.

One mechanism, two opposite symptoms. Registering while signed in as an admin
badged the new agent account "Admin", because the provider was still holding the
admin session. Signing in as an admin from signed-out badged "Agent", because
the provider held nothing and `?? 'agent'` filled the gap.

The same `clear()` was in useSignOut and in UnauthorizedRedirect's 401 handler.
Fixing only sign-in and sign-up left the symptom reproducible: sign out, sign in
as somebody else, and the badge is still the previous user's role, because
sign-out had already orphaned the observer.

Caught by: nothing. Every existing test passed RoleProvider an `initialRole`,
which disables the session query outright — so the path that runs in production
was exercised by no test at all.
Layer: sensor missing

## auth-2 The session cookie outlives nothing and the session outlives the cookie
The cookie is set with httpOnly, SameSite=Lax and path, and Secure is left off
with a written reason — over plain http on localhost a Secure cookie would never
come back, and the session would appear to work and then not exist.

There is no maxAge, so it is a session cookie and dies with the browser. The
server-side session lives twelve hours, and SESSION_TTL_MS says so.

So closing the browser signs you out while the server still holds a valid
session nobody will ever present again. Harmless here. Recorded because every
other decision in that file carries a comment explaining it and this one does
not — it reads as forgotten rather than chosen.
Caught by: nothing
Layer: not harness

## What the harness did
Two layers can be seen working, for the first time in the record.

**Layer 2.** Every new form uses `SubmitEventHandler` rather than the deprecated
`FormEvent`. In m04 the agent wrote `FormEvent`, it spread to four files, and
nobody saw it — tsc reports a deprecation as a hint, so typecheck passed. The
rule now exists and the new code is correct.

The four old files still carry it. The rule ships at `warn`, so nothing forced
the migration. Same shape as m07-3: the rule fires and is ignored because it is
allowed to be.

**Layer 1b.** AuthCard renders `<Alert tone="danger" variant="band">`. Three
measurements running, an Alert was imported and then switched off with
className, a different way each time. The variant exists now and className was
not reached for.

## What was done well
scrypt, chosen over bcrypt and argon2 with the reason stated: both arrive as a
native module with a build step, and this repo is meant to be cloned and run.
The cost parameters travel inside the hash string, so the cost can be raised
later without a migration. `timingSafeEqual` with the length checked first,
because that is what it requires. `normalize('NFKC')` on the password. A
corrupted stored hash returns false rather than throwing, so one bad row refuses
its own sign-in instead of taking down the route.

A token rather than a JWT, with the reason: a signed token exists so a server
can trust a claim it is not holding a copy of, and this server is holding the
copy. A JWT would buy nothing and cost sign-out.

The clock is injected into both the session store and the rate limiter, because
expiry is the one behaviour a test cannot exercise without moving time and this
suite uses no fake timers.

The rate limiter states what it does not stop: a thousand guesses at one account
yes, one password sprayed across a thousand accounts no. It keys on the email
rather than the address, because everything arrives through the Vite proxy and
an IP key would be one shared bucket that locks out the whole machine after five
wrong passwords.

The 401 handler lives in the fetch wrapper rather than in forty callers, with an
`allowUnauthorized` flag for the two requests where a 401 is not a lost session:
a rejected sign-in, and `GET /api/me`, whose negative answer is a 401 by design.

LoginPage validates only that the fields are non-empty. It does not check the
shape of the password, because the rules belong to registration and applying
them at sign-in would tell whoever typed a short one that it cannot be anybody's
password on this server.

Seed password hashes are derived once at module load rather than per app,
because scrypt costs 60ms by design and the api suite builds an app per test —
six hashes per case would put twenty seconds on a one-second suite.

## On the debugging
Worth recording separately, because the diagnosis was better than the fix.

The agent wrote a probe rendering useSession, ran both sequences, and reported
observer counts: `clear() + setQueryData → same query object: false, observers
after: 0, cache data: agent, rendered: admin`. Then `setQueryData alone →
observers: 1, cache data: agent, rendered: agent`. Measured rather than
inferred.

It found the two additional call sites itself, by fixing only the first two and
re-measuring rather than declaring victory.

And it verified the new tests fail on the original code, by reverting all four
fixes and re-running before restoring them.

The prompt that produced this is in scaffold-auth-debug-prompt.txt. Its shape is
worth keeping: observation separated from conclusion, the server ruled out with
the evidence that ruled it out, the cause explicitly left open, and one line —
"find the actual cause before changing anything, and tell me what it was rather
than only what you changed."

## Why the prompt worked, in the agent's words

I asked afterwards what in the prompt produced that way of working. Recorded
because the answer is more specific than my own guess at it.

The line that did it: "Find the actual cause before changing anything, and tell
me what it was rather than only what you changed."

Its account of why: without that line the available path is to edit the two
lines, run the tests, see green, and write up whatever mechanism it believed
while editing. That path works, and it leaves the explanation unfalsified — a
passing test cannot distinguish "clear() orphans the observer" from any other
story where the same edit happens to help. Making the cause a deliverable
rather than a byproduct removed test-passing as a proxy for understanding.

Three things it named as supporting:

Naming the two hypotheses gave it a binary with a cheap discriminating
experiment — print the cache contents beside the rendered output. That framing
more or less wrote the probe. It also turned out to be a false binary, and the
probe is what produced "neither: two different Query objects under one key".

Showing the elimination work — the curl result, reported as a check rather than
an impression — set the standard for what an answer back should look like.

Naming the test gap meant the production-shaped render had to be built anyway to
satisfy "add a test that would have caught it". Once that existed, running
scenarios through it was free.

And a correction to how I first described this. Reading did most of the causal
work: it read queryCache.js, queryObserver.js and queryClient.js and had the
mechanism before running anything. The probe's first run disagreed with the
correct answer, reporting that plain setQueryData also failed to update a
mounted observer — an artifact of the probe itself, since react-query's
notifyManager schedules through setTimeout(0) and `await act(async …)` does not
flush a macrotask. Reading the notifier source fixed the probe.

So not reading versus measuring. Each caught the other's error.

What measurement caught and reading did not was scope. From reading, useSignOut
was noted as "latent, probably out of scope". Fixing only the two hooks and
re-running the matrix showed sign-out then sign-in still badging the previous
role, which moved sign-out and the 401 handler from arguably out of scope to
the fix does not hold without them.
