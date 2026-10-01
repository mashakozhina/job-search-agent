# Job Search Agent (Claude skill)

A Claude skill that turns Claude into a disciplined personal job search agent. It searches company careers pages, ATS platforms, job boards and LinkedIn, **verifies every link on the company's own page before recommending it**, filters by your role, seniority, languages, locations and salary floor, and fit-checks job descriptions against your CV.

It works for any role. Everything personal lives in one config file.

## What it does

- **Job search (Mode 0)**
  - Weekly cadence: the first search of the week is a full **BACKFILL**; later sessions are short **DAILY** check-ins that only surface new postings.
  - Multi-source: Google Jobs, the main ATS platforms (Greenhouse, Lever, Ashby, Workday, SmartRecruiters and more), niche and remote boards, LinkedIn recruiter posts, and a rotating watch-list of company careers pages.
  - Strict verification: a role is only recommended when its job description has been opened on the company's careers or ATS page. Aggregator links, 404s and JavaScript-only pages are listed separately as "leads to check yourself".
  - Honest scoring: fit 1–10, GO/SKIP, and no padding when there's little new.
- **Tracker (Mode 2)**: tell Claude "I applied to X", "X rejected me" or "interview with Y on Friday" and it updates your application tracker (a Notion database) and confirms in one line. Job search then skips roles you've already applied to.
- **Fit check (Mode 1)**: paste a job description or link to get a fit score, the exact ATS keywords to use, which CV sections to tailor, red flags and a GO/NO-GO call.

## Folder layout

```
job-search-agent/
├── SKILL.md                     # agent logic (what Claude reads first)
├── config.template.yaml         # copy to config.yaml and fill in
├── project-instructions.template.md  # short text for your Claude Project
├── references/
│   ├── search-strategy.md       # sources, query templates, credit control
│   ├── verification-rules.md    # what counts as a live, verified role
│   ├── output-formats.md        # shortlist, leads, recruiters, daily briefing
│   └── tracker.md               # application tracker (Notion) rules and schema
└── examples/
    └── barcelona-qa/config.yaml # complete example: Senior QA Automation / AI QA in Barcelona
```

## Quick start (about 10 minutes)

### 1. Download the skill

Clone the repo, or click **Code → Download ZIP** on GitHub and unzip it:

```
git clone https://github.com/<owner>/job-search-agent.git
```

### 2. Create your config

Copy `config.template.yaml` to `config.yaml` in the same folder and fill it in. The fastest route is to start from the worked example:

```
cp examples/barcelona-qa/config.yaml config.yaml
```

Then change the parts that are about you:

| Section | What to set |
|---|---|
| `candidate` | CV file name, one-line positioning, languages you can work in |
| `roles` | Role types you want and don't want, plus 2–3 title sets of 3–4 exact job titles each |
| `seniority` | Levels you accept and levels to always skip |
| `locations` | Your home scope, other scopes to offer, remote/hybrid/on-site, relocation cities |
| `salary` | Global floor and optional per-city floors |
| `sources` | Job boards for your country, region and role |
| `companies` | Optional watch-list of companies to check directly, split into up to 6 daily batches |
| `tracker` | Where your applications are tracked: Notion (Claude updates it), a file (read only) or none |
| `exclusions` | Companies to always skip |

Not sure what to put? Install the skill first, then say **"help me fill in my config"** and Claude will ask you one question at a time.

### 3. Install the skill

**Claude app (web, desktop, mobile)**
1. Make sure code execution is on: **Settings → Capabilities** (Free, Pro, Max). On Team/Enterprise plans an admin controls this.
2. Put your `config.yaml` inside the `job-search-agent` folder (or upload it to your Claude Project instead, step 4).
3. Zip the **whole** `job-search-agent` folder, so the zip contains `job-search-agent/SKILL.md`.
4. Go to **Customize → Skills**, click **+ → Create skill → Upload a skill**, and choose the zip.
5. Make sure the skill is toggled on.

**Claude Code**
- For all your projects: copy the folder to `~/.claude/skills/job-search-agent/`
- For one project only: copy it to `.claude/skills/job-search-agent/` in that project
- Invoke it with `/job-search-agent`, or just say "job search".

### 4. Create a Claude Project and add a short instruction (required)

Installing the skill is **not enough on its own**. The skill holds the rules, but Claude still needs to know where your CV, config and tracker are, and a plain "hi" won't reliably open a skill. A Project fixes both.

1. In Claude, create a new **Project**, e.g. "Job search".
2. Upload to the Project's files:
   - your CV (the file name must match `candidate.cv_file`)
   - your `config.yaml`, if you didn't put it inside the skill zip
   - optionally, an application tracker file, if you don't use Notion (see **Tracker** below)
3. Open `project-instructions.template.md`, fill in the placeholders, and paste it into the Project's **instructions**. It's about ten lines. Don't paste the whole skill there: the rules already live in the skill, and two copies drift apart.
4. Always start your job search chats **inside this Project**.

### 5. Turn on web access

The agent needs **web search** (and web fetch) enabled in the chat, otherwise it can't search or verify links.

## How to use it

Start a new chat **inside your job search Project** and say **"hi"**. Claude offers two modes:

### Job search

Say **"job search"** (or "find jobs", "daily search", "morning briefing").

- The first search of each week (Monday–Sunday) is a **BACKFILL**: a full sweep of every source. Expect it to take a while.
- Every later search that week is a quick **DAILY** check that only shows new postings, plus one batch of your company watch-list.
- It starts in your home scope. At the end, it offers a wider one (country, region remote, worldwide remote or relocation). Say "yes" to run it.
- You can override on the spot: "job search, EMEA remote", or "run a backfill".

What you get back:
1. **Shortlist**: only roles scoring 7+/10 whose job description Claude opened on the company's own careers or ATS page. Each has a "Verified:" line.
2. **Leads to check yourself**: roles it couldn't verify (aggregator-only, JavaScript-only pages), with the reason.
3. **Recruiters actively hiring**: LinkedIn "I'm hiring" posts, with a suggested outreach line.

Tell it what to skip ("applied to X", "not interested in Y") and it excludes those for the rest of the chat.

### Fit check

Say **"fit check"** and paste a job description or link. You get:
- a fit score out of 10 and a one-line verdict
- whether the link is still live
- the exact keywords from the job description to use in your CV
- which CV sections to tailor
- red flags (seniority, language, location, salary, visa)
- a GO / NO-GO recommendation

### Tracker

Connect Notion in Claude (Settings → Connectors) and say **"set up my tracker"**. Claude creates a *Job Applications* database with a pipeline board and puts its link in your config. From then on, just tell Claude what happened:

- "I applied to Acme, Senior QA Automation Engineer"
- "Acme invited me to a technical interview on Friday"
- "Acme rejected me after the final round"
- "What's in my pipeline?"

No Notion? Set `tracker.type: file` to use a spreadsheet or PDF in your Project (Claude reads it but can't edit it), or `none`.

### Example prompts

```
hi
job search
job search, Spain-wide
run a backfill this week
fit check: <paste job description>
fit check https://jobs.lever.co/company/1234
skip Company X, I've already applied
I applied to Company Y, QA Engineer
what's in my pipeline?
help me fill in my config
```

## Privacy

`config.yaml`, CVs (`.docx`, `.pdf`) and tracker files are in `.gitignore`. Keep your contact details, CV and application history out of any fork you publish.

## Maintenance

Companies move between ATS platforms. Re-check the `companies` lists in your config about once a quarter.

## Limitations

- Search coverage depends on what web search returns; some careers sites block automated fetching or need JavaScript, so those roles are reported as unverified leads.
- Salary conversions use the exchange rate Claude finds on the day.
- The agent gives information to help you decide. It doesn't apply to jobs for you.

## Licence

MIT — see [LICENSE](LICENSE).
