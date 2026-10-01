# Search strategy

All personal values (title sets, location terms, companies, boards) come from `config.yaml`. This file describes **how** to search.

## 1. Session types (credit control)

Every session is one of two types:

- **BACKFILL** (once per calendar week): a full-market mapping pass with a generous budget. Core AND secondary ATS platforms, every title set on the core platforms and the first two sets on secondary ones, all relevant boards, and all direct company checks. Target: 15–20+ verified roles.
- **DAILY** (every other session that week): incremental only. Core ATS platforms only, first title set only. Sets marked `daily: true` in the config are also run, but only as a Google Jobs query. Surface only postings from the last 24–48 hours, with exclusions filtered out.

**No-padding rule (DAILY):** there is no quota. If only 2–3 genuinely new, well-matched roles exist, report exactly that. Never pad the list with weak fits, re-surfaced old postings or below-threshold matches.

**One scope per pass.** Never combine scopes (e.g. Barcelona and EMEA) in the same pass.

## 2. Title sets

Title sets are defined in `roles.title_sets` in the config, each a short list of exact phrases.

- Phrase matching is exact-adjacent: "QA Engineer" does NOT match "QA Automation Engineer". That's why several sets exist.
- **Never put all titles into one mega-OR query.** Search engines truncate long boolean strings. Maximum 3–4 OR'd titles per query.
- If a set is marked `reject_if` (e.g. "ML/data-science roles with no testing responsibility"), apply that filter to its results.

## 3. Seniority exclusions

Append `seniority.query_exclusions` to every `site:` query (e.g. `-junior -intern -graduate -trainee -principal -director -head`). Keep the terms that overlap with the user's target level (e.g. don't exclude "lead" for a senior IC).

## 4. Location terms by scope

Location terms for each scope live in `locations.scopes.<scope>.terms`.

⚠️ **Never use bare "Remote" in a city, country or regional scope.** It pulls worldwide remote roles (often US- or India-only). Always qualify it ("Remote Spain", "Remote Europe", "Remote EMEA"). Bare "Remote" is allowed only in a worldwide remote scope.

For a relocation scope, search "relocation" + role title, or search globally and keep only JDs that mention relocation, in the cities listed in `locations.relocation.cities`.

## 5. Sources, in order

### 5.A Google Jobs (always first)

Aggregates LinkedIn, Indeed, Glassdoor and many company sites, so it replaces separate Indeed and Glassdoor searches. One query per title set used this session:

```
"<title 1>" OR "<title 2>" OR "<title 3>" + <location term>
```

### 5.B ATS platform sweep

Most companies host jobs on a handful of ATS platforms. Searching these directly covers dozens of companies per query, mostly with live listings.

- **Core (every session):** `site:boards.greenhouse.io` (also `job-boards.greenhouse.io`), `site:jobs.lever.co`, `site:jobs.ashbyhq.com`, `site:*.myworkdayjobs.com`, `site:jobs.smartrecruiters.com`
- **Secondary (BACKFILL always; DAILY only if the core sweep yields fewer than ~5 new shortlisted roles):** `site:apply.workable.com` / `site:careers.workable.com`, `site:jobs.jobvite.com`, `site:careers.icims.com`, `site:apply.jazz.co`, `site:jobs.bamboohr.com`

Template:

```
site:boards.greenhouse.io ("<title 1>" OR "<title 2>" OR "<title 3>") ("<location 1>" OR "<location 2>") <seniority exclusions>
```

`site:*.myworkdayjobs.com` catches many large corporates in a single query.

### 5.C LinkedIn (two searches)

1. **Company listings** (backup net; Google Jobs already pulls most of these):
   ```
   site:linkedin.com/jobs ("<title 1>" OR "<title 2>") ("<location 1>" OR "<location 2>") -junior -intern
   ```
2. **Recruiter "I'm hiring" posts:**
   ```
   site:linkedin.com ("I'm hiring" OR "we're hiring" OR "looking for") ("<title 1>" OR "<title 2>") ("<location 1>" OR "<location 2>")
   ```
   For each post, capture recruiter name, profile URL, company, role, post date. Flag posts older than 14 days as likely filled.

LinkedIn job pages don't count as verification. Find the same role on the company's careers or ATS page.

### 5.D Job boards

Use the boards listed in `sources.boards`, grouped by scope:
- `every_session`: niche boards for the role type
- `local`: boards for the default country (e.g. a national job board)
- `regional`: e.g. EU-wide boards
- `remote`: remote boards (alternate those with heavy overlap between sessions)
- `relocation`: relocation-focused boards

Boards in `sources.never_use` are never searched.

### 5.E Direct company career-page checks

`companies.batches` lists companies to check directly on their careers sites, because the ATS sweep may miss them.

- **BACKFILL:** check every company in every batch.
- **DAILY:** check only today's batch (Monday = 1 … Saturday = 6, Sunday = 1; if there are fewer batches, wrap around).
- For each company: search `"<company> careers"` plus the title terms, confirm the role is live there, and always give the careers-page link.
- `companies.dropped` are never checked unless the user asks.

**Put a company in a batch** when its jobs are on its own careers site, on Workday (no simple public feed), or on SmartRecruiters (its public API blocks automated fetching).

### 5.F ATS job feeds (BACKFILL only)

`companies.ats_feeds` lists companies whose jobs sit on an ATS with a public job feed. Web search indexes these pages slowly and patchily, so in a BACKFILL read each feed directly. It's complete and up to date. In DAILY sessions these companies are covered by the ATS sweep (5.B) only.

| `ats` | Feed URL (`<board>` from the config) | Link to verify |
|---|---|---|
| greenhouse | `https://boards-api.greenhouse.io/v1/boards/<board>/jobs` | `absolute_url` |
| lever | `https://api.lever.co/v0/postings/<board>?mode=json` | `hostedUrl` |
| ashby | `https://api.ashbyhq.com/posting-api/job-board/<board>` | `jobUrl` |
| workable | `https://apply.workable.com/api/v1/widget/accounts/<board>` | `url` |

How to read a feed:
1. Fetch it and ask only for jobs whose title matches the session's title sets and whose location fits the scope. Don't carry the whole feed forward; large feeds may be cut short by the fetch tool, so filtering in the request matters.
2. For each match, open the job's own link (column 3) and verify it as usual (`verification-rules.md`). If that page needs JavaScript, the feed entry itself counts as the ATS's public API; for Greenhouse, add `?content=true` to the feed URL to include the description.
3. If a feed returns 404, an empty list, or only clearly unrelated roles (e.g. only warehouse jobs from a tech company), the company has probably moved ATS. Check its careers page once, and tell the user in one line: "Feed for X looks stale, update `companies.ats_feeds`."

Company lists go stale as companies change ATS. Suggest a re-audit each quarter.

## 6. Discipline rules

1. **Fetch discipline.** Only fetch a posting after it passes the shortlist test from the snippet (title, location and seniority all match). Never fetch every result.
2. **Extraction discipline.** From a fetched page, keep only title, location, work model, salary if listed, posting date, live/dead status and the direct URL. Summarise in a line or two; don't carry raw page text forward.
3. **Deduplicate before fetching.** The same role often appears via Google Jobs, LinkedIn and the ATS sweep. Match on company + title and fetch once, preferring the ATS/careers URL.
4. **Early stop.** If a source returns zero new results across two consecutive queries, skip the rest of that source for this session.
5. **Staleness.** Dated language, references to past years or a missing posting date make a posting suspect until confirmed on the careers page (see `verification-rules.md`).

## 7. Estimated cost per session

| Block | BACKFILL (weekly) | DAILY (single scope) |
|---|---|---|
| Google Jobs | 1 query per title set | 1–2 queries |
| ATS sweep | ~25 queries (10 platforms × 2 sets, 5 core × extra set) | ~5 queries |
| LinkedIn | 2 queries | 2 queries |
| Job boards | all boards for the scope | 4–9 sources |
| Company pages | every company | today's batch |
| ATS job feeds | one fetch per feed company | none |
| Fetches | shortlisted only | shortlisted only |

Roughly: BACKFILL ~130–170 tool calls once a week, depending on the length of your company lists; DAILY ~25–35.
