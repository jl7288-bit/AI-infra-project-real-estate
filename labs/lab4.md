# Labs 3 + 4 · Pipeline submission

**Name:**
**Due:** Sunday, November 22 at 11:59 PM

## Links

**A successful run of your pipeline** (Actions tab → the green run → copy the address):
https://github.com/jl7288-bit/AI-infra-project-real-estate/actions/runs/45121149757
**A deliberately failed run** (Run workflow with "Break the data on purpose" ticked → the red run → copy the address):
https://github.com/jl7288-bit/AI-infra-project-real-estate/actions/runs/45130610994

**Your table on MotherDuck:** `course.main.____________`
course.main.manhattan_parcels_clean
course.main.manhattan_far_comparison
course.main.manhattan_far_summary
course.main.manhattan_far_ranking
## Reflections

Answer in a few sentences each. Specific beats long.

**1. Your checks.** Which of your checks would you be most worried to see fail, and what would go wrong for a user of your app if bad data got through?
I would be most concerned if the BBL uniqueness check failed. Each BBL should identify one parcel, so duplicates could inflate parcel counts and distort the FAR screening results. They could also create incorrect matches when I connect PLUTO to ZAP later. These results are preliminary screening indicators, not confirmed development capacity.
**2. Run it twice.** What happens to your MotherDuck table if your pipeline runs twice in one day? How do you know?
The pipeline uses create or replace table, so a second successful run replaces each published table instead of appending duplicate rows.
**3. Your schedule.** How often does your source actually update? Is daily the right schedule for it, and would you take a full copy or only what's new?
NYC Open Data lists PLUTO’s update frequency as quarterly, so downloading the entire dataset daily is more frequent than necessary. The daily schedule is useful for practicing automation in this lab. For my product, I would check for source updates weekly and refresh the full Manhattan snapshot when a new version becomes available. A full refresh captures changes to existing parcels as well as additions and deletions.
**4. What leaves your control.** Next week an AI model will see your table's description and users' questions. Is there anything in your data, or anything a user might type, that shouldn't be sent to a free AI service? Why or why not?
The selected PLUTO fields are public property data, but users’ questions could contain confidential acquisition plans, budgets, client information, or unpublished deal details. Those details should not be sent to an external AI service without appropriate permission and privacy review. I would send only the necessary context and remove sensitive information. MotherDuck tokens, passwords, and other credentials should never be included in prompts.
---

## Set up your secrets (once)

Your MotherDuck token is a password. It goes in two secret stores, **never** in a file, a notebook, or an AI chat.

**Get a token:** app.motherduck.com → your name (bottom left) → **Settings** → **Access Tokens** → **Create token**. Name it `codespaces`. Copy it.

**A. For your Codespace** (so `python pipeline.py` can publish):
github.com → your profile picture → **Settings** → **Codespaces** → **New secret**. Name `MOTHERDUCK_TOKEN`, paste the token, choose your course repo under **Repository access**, **Add secret**. Then restart your Codespace.

**B. For GitHub Actions** (so the daily run can publish):
your course repo on github.com → **Settings** (repo tab, not your profile) → **Secrets and variables** → **Actions** → **New repository secret**. Name `MOTHERDUCK_TOKEN`, paste the same token, **Add secret**.

If a token ever leaks, delete it in MotherDuck right away and make a new one (then update both secrets).

---

**Checklist before you submit**

- [ ] `project.toml` points to **your** API, with at least five checks of your own (each with a why) and at least three tables: one clean table and at least two analysis tables (trend, key metrics or ranking)
- [ ] `docs/findings.md`: three findings, each backed by one of your tables
- [ ] A green run and a red run on the Actions tab, both linked above
- [ ] `docs/data_dictionary.md` filled in for every table you publish
- [ ] This file answered
- [ ] No token anywhere in your files (tokens start with `eyJ`: search your repo with Cmd+Shift+F)
- [ ] Committed and pushed; repo URL submitted on CourseWorks
