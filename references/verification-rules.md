# Verification rules (mandatory — no exceptions)

These rules exist because job boards and aggregators are full of stale and closed listings. A dead link wastes the user's time and trust.

## What counts as verified

Fetch the job posting URL directly, preferably the company's own careers page or ATS page (Greenhouse, Lever, Ashby, Workday, SmartRecruiters, Workable, etc.).

A role is **verified** only when the fetched page shows the job description itself: the title plus responsibilities or requirements.

These do **not** count as verification:
- a 404 or any other error
- a redirect to a general jobs list or an `?error=true` page
- a blank page that needs JavaScript ("You need to enable JavaScript")
- a job-board or aggregator page (Remotive, Himalayas, Built In, Glassdoor, Jooble, TestDevJobs, freehire, Indeed, LinkedIn guest pages and similar)
- a search-engine snippet

## If the page needs JavaScript

Try once to verify another way:
- the ATS's public API or JSON feed (e.g. `boards-api.greenhouse.io/v1/boards/<company>/jobs`), filtering to the relevant titles rather than dumping the whole feed;
- a server-rendered version of the careers page;
- the company's own job listing on a different ATS board.

If that fails too, the role is **unverified**.

## Unverified roles

- Can never be GO.
- List them only under "Leads to check yourself", with the reason (e.g. "careers page needs JavaScript — couldn't confirm").
- Fit score capped at 6/10.
- If the only link that works is an aggregator's, label it "aggregator link — may be stale".

## Freshness

- "Reposted X days ago" or "5d ago" on an aggregator is **not** evidence that a role is live.
- Flag roles older than 21 days.
- Skip roles older than 30 days unless the careers page confirms they're still open.
- A posting with dated language, references to past years, or no posting date is suspect until confirmed on the careers page.

## Location and eligibility

- Read the location line on the verified page. "Remote" often means one country or region only (e.g. US only, LATAM only). Skip if the user isn't eligible.
- For relocation roles outside the user's work-authorisation area (`locations.relocation.work_authorisation`), visa sponsorship must be stated on the company's page or confirmed by the company. Otherwise skip.
- Note any "one application per group" rules (some company groups don't allow parallel applications).

## Reporting

Every GO role must include:

> **Verified:** [date] on [careers page / ATS page], showing the job description.

If this line can't be filled in honestly, the role is not GO.
