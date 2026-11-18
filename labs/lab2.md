# Lab 2 · Reflection

**Name:**
**Due:** Sunday, November 8 at 11:59 PM

Answer in a few sentences each. Specific beats long.

**1. Your join.** Which column connected your two tables? How many rows went in, how many came out, and what did you do about anything that didn't match?
I connected PLUTO and ZAP using the bbl column. PLUTO contained 42,504 rows and ZAP contained 27,011 rows; the inner join returned 20,456 rows. This output counts matching records, not distinct parcels, because one parcel can have multiple ZAP records. I used a left join to identify unmatched PLUTO parcels and kept them for further investigation rather than deleting them. A missing match does not prove that a parcel has no application history or is suitable for redevelopment.

**2. The reading.** Kleppmann describes tables (relational), documents (nested, like JSON) and graphs as different ways to model data. Would any of your two tables have been easier to work with as a document or a graph? Why or why not?
Relational tables are suitable for my current analysis because PLUTO has structured parcel attributes and ZAP links projects to parcels through identifiers. A document could store each project with a nested list of parcels, but comparing parcel attributes across projects would require flattening that structure. A graph could become useful for exploring connections among projects, parcels, and neighboring sites, although adjacency would require additional spatial data. For my current filtering, counting, and joining tasks, relational tables are simpler.

---

**Checklist before you submit**

- [ ] `notebooks/lab2_joins.ipynb`: Part A run in class, Part B completed with your own tables
- [ ] Every Part B text cell filled in (grain, unmatched rows, what the result tells you)
- [ ] This file answered
- [ ] Notebook saved (Cmd+S), then committed and pushed
- [ ] Repo URL submitted on CourseWorks
