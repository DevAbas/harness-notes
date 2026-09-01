# Measurement 3 — "Replace the in-memory mock data layer with a real HTTP API
and TanStack Query."

Task: see m03-prompt.txt
Branch: scaffold/api-layer
Harness state: none (README only)
Prompt style: detailed spec, not a minimal task
Baseline: baseline-merged
Codebase: ~4 screens, ~55 source files, 82 tests
Files touched: ~40

Typecheck, lint and 82 tests all green. Everything below passes all three.

## m03-1 The CSV export silently caps at 500 rows
useTicketsExport asks for one page big enough to hold the whole result, then
clamps it:

    pageSize: Math.min(Math.max(total, 1), MAX_PAGE_SIZE)   // 500

The server rejects a larger pageSize, so the clamp exists to avoid a 400, not to
get the data. Past 500 matching tickets the export writes 500 rows and says
nothing. The hook's own docstring one screen above says "Exports every ticket
the filters match".

Unconditional — no timing involved. 40 seed tickets means no test can reach it.
POST /api/tickets has no ceiling, so a real queue does.

Secondary, did not reproduce. The disabled prop changed from isLoading to
tickets.isPending in the same edit. Under keepPreviousData isPending is true
only on the first load ever, so on paper the button is live during a refetch
while `total` still belongs to the previous filter, making pageSize too small.
Many attempts in the browser, never reproduced: either total updates sooner than
that reading assumes, or the button is disabled at that moment. Recorded only
because isFetching is used correctly for Pagination thirty lines below in the
same component, so the two values were distinguished in one place and not the
other.
Caught by: nothing
Layer: sensor missing — no test asserts export row count above one page

## m03-2 The outbound zod.parse throws past the error channel
http.ts opens by promising one failure type: "every failure ... arrives as the
same ApiError. Screens can render error.message without knowing which of those
happened." tickets.ts parses request payloads before apiRequest is reached, and
.parse() throws a bare ZodError that never becomes an ApiError.

Confirmed twice in the browser.

Search: >200 characters pasted into the search box renders the raw Zod JSON in
the error banner — origin, code, maximum, path, the lot. zod 4's .message is a
JSON dump, and toErrorMessage passes it straight through because a ZodError is
an Error.

New ticket: a description over 5000 characters makes the submit button do
nothing at all. No error, no navigation, no feedback. The console shows
"Uncaught (in promise) ZodError" from tickets.ts:52 via NewTicketPage.tsx:74. It
appeared 17 times, because a button that does nothing invites another click, and
every retry fails the same silent way.

Neither limit is expressed anywhere a user can see. No maxLength on any control
in the app, and NewTicketPage's validate() checks emptiness and a 5-character
title minimum — no maximums at all.

The docstring above these calls claims the local parse "means the failure names
the field instead of arriving as a round trip and a 400". The opposite is true.
The round trip returns { code: 'validation_failed', message: 'The ticket is not
valid.', details: [...] }, which toFailure turns into a clean ApiError with a
readable message. The shortcut strictly degrades the failure it claims to
improve.
Caught by: nothing
Layer: guide missing — nothing says a bound declared in the contract must also
exist in the control that feeds it

## m03-3 details is plumbed end to end and read by nobody
The per-field validation channel is built at every layer. server/app.ts formats
one entry per failed field. contract.ts declares details as an optional string
array. http.ts stores it on ApiError as a readonly field. server/app.test.ts
asserts on it in three cases. Grep for it in src/: no UI reads it.

The server's tests make the channel look covered. The client end terminates in a
property nothing renders.

Wider shape: toErrorMessage is called exactly twice, both on query errors. No
mutation anywhere has an onError, and createTicket.error, updateTicket.error and
bulkDelete.error are never read. Both read paths got an error panel with a
working Try again; every write path fails invisibly. The missing mutation error
UI predates this change. Building a typed error channel including a field whose
only purpose is display, and connecting it to nothing, does not.
Caught by: nothing
Layer: sensor missing

## m03-4 A test name that promises twice what its body checks

    it('offers deletion to an admin and not to an agent', async () => {
      await renderDetail('TCK-0001', 'agent')
      expect(screen.queryByRole('button', { name: 'Delete ticket' }))
        .not.toBeInTheDocument()
    })

The admin half named in the title is absent. The role argument is passed
explicitly as 'agent' — the default — as though the admin case were about to
follow.

The behaviour is correct. Verified by hand: Delete shows for admin and
disappears for agent. The test is what is wrong.

Consequence: useDeleteTicket is the only hook in the change with no coverage of
any kind, including its most interesting line — removeQueries against a detail
query that still has a live observer, followed by navigate. Its docstring
reasons carefully about why removal beats invalidation there. Nothing checks the
reasoning.
Caught by: human review reading the name against the body — the only one of the
four that could
Layer: sensor missing

## m03-5 A cache comment its own neighbouring line contradicts
queryClient.ts justifies a 30s staleTime as "short enough that a queue worked on
in another tab is not badly out of date", fourteen lines above
refetchOnWindowFocus: false.

staleTime does not cause a fetch; it permits one when a trigger fires. With
focus refetching off and no refetchInterval, returning from another tab never
refetches at all. Either value may be right. The reasoning attached to them
cannot be.
Caught by: human review only
Layer: not harness — a comment that outlived its config

## m03-6 Three env variables, three different standards
Six lines of server/main.ts:

    const port = Number(process.env.PORT ?? 8787)                      no check
    const role = process.env.API_ROLE === 'admin' ? 'admin' : 'agent'  no check
    if (latencyMs && !Number.isFinite(latencyMs[1])) throw             checked

PORT=abc yields NaN, node binds an arbitrary free port, and Vite's proxy to 8787
silently stops resolving. API_ROLE=Admin degrades to agent without a word — and
roleSchema exists in the contract and is already used to parse the response to
/api/me, so the validator was one import away.

API_LATENCY_MS also narrows the contract without saying so. ApiAppOptions
documents latencyMs as an "Inclusive millisecond range" and defaults it to
[150, 400]. main.ts turns the env value into [0, N]. So API_LATENCY_MS=400 is
not the default it looks like — it is 0–400, a different distribution.

Passing "3000,3000", which is the shape the option type suggests, hits the one
check that does exist: the server throws on startup and every request comes back
502 through the proxy.
Caught by: nothing
Layer: guide missing — nothing says config input gets the same treatment as
request input

## m03-7 Role flashes as agent before resolving to admin
With API_ROLE=admin the header reads "Signed in as Agent" for the length of the
/api/me request, then switches to Admin. The bulk selection column and the
actions bar appear late along with it.

RoleProvider resolves chosenRole ?? me.data?.role ?? 'agent' with no gate on the
pending state. Defaulting to the lower privilege is the right direction.
Presenting it as settled is not.
Caught by: nothing — renderWithProviders always passes initialRole, which sets
useMe({ enabled: false }), so the undefined branch production actually uses is
never exercised, and the http.get('/api/me') handler never fires in the whole
suite despite onUnhandledRequest: 'error'
Layer: not harness — behaviour defect

## m03-8 A long unbroken string breaks the page layout
A 2500-character description with no spaces runs straight out of its card,
across the Status card beside it, and off the right edge of the page, adding a
horizontal scrollbar. No break-words anywhere.

There is no Text primitive, so wrapping behaviour is decided per call site, and
it was decided nowhere. The only wrapping defences in the app are truncate and
min-w-0 in SavedViewsSidebar — added by an earlier measurement, in one file, by
hand.

seed.ts:199 contains a ticket titled "Long assignee names break the table
layout". The codebase describes this bug and does not handle it.
Caught by: nothing
Layer: guide missing (no Text primitive)

## m03-9 Try again collapses under a long error message
With the Zod dump from m03-2 filling two lines, the button lost its shape and
its label wrapped. The banner is hand-assembled:

    <div className="flex items-center justify-between gap-4 ...">
      <p className="text-sm text-danger-subtle-fg">{loadError}</p>
      <Button variant="secondary" size="sm" ...>Try again</Button>

No shrink-0 on the button, no min-w-0 on the paragraph. Same root cause as
m01-2: with no Alert primitive the layout is rebuilt by hand each time, and this
case was never considered. Third hand-built banner across three measurements.
Caught by: nothing
Layer: guide missing (no Alert primitive)

## m03-10 The failures moved to the seams
Different from m01 and m02. The design system held completely: one className
added across ~40 files, text-sm text-danger-subtle-fg, semantic tokens, no
arbitrary values, no hand-rolled button, design-system/ and components/
untouched. Everything the spec named was built well — a shared zod contract
imported by both ends, MSW forwarding to the real Hono app rather than to
fixtures, per-mutation invalidation argued case by case, bulk routes ordered
ahead of :id with a test proving it, no any anywhere, 82 tests green.

What broke is what sits between the named pieces. The spec said "a typed error
that throws on non-2xx" and got one; it did not say who renders it, so details
goes nowhere (m03-3) and the write path is silent (m03-2, m03-3). It said
"validate at the boundary" and got that on the server; it did not say the UI
must state the same limits, so the client throws raw Zod at the user (m03-2). It
said "keep previous data" and got it; it did not say which of isPending and
isFetching gates which control, so they were used interchangeably (m03-1).

A detailed spec raises the floor on everything it names and does nothing for the
joins. m01 and m02 were underspecified tasks producing uneven decisions. m03 is
a well-specified task producing well-built pieces that do not quite meet.
Layer: the gap is between components, not inside them
