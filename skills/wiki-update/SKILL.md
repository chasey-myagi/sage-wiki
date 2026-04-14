---
name: wiki-update
description: >
  Revise existing wiki pages when new session evidence contradicts or supersedes
  prior understanding, or when the Agent discovers during a task that a wiki page
  is outdated or incorrect.
  Trigger: "wiki is wrong about X", "update wiki page", "the wiki says X but
  actually Y", "correct wiki", "wiki update", "this contradicts [[slug]]",
  "[[slug]] is outdated".
  Also trigger automatically during task execution when the Agent observes behavior
  that contradicts a wiki page it just applied — don't silently carry on with wrong
  knowledge.
user-invokable: true
args:
  - name: page
    description: The wiki page slug to update (optional — skill will identify it if not specified)
    required: false
---

# wiki-update — Revise Wiki Pages

Updates one or more wiki pages when new evidence from the current session
contradicts, supersedes, or refines prior accumulated knowledge. Keeps the
wiki accurate as the domain evolves.

Every change requires a source — this keeps the wiki auditable.

## Step 0 — Load context

Read `SCHEMA.md`. Read the specific page(s) to be updated in full.
Do not propose changes from the index alone — always read the full page first.

## Step 1 — Identify what needs to change

Determine the trigger:
- **Agent observation** — Agent applied wiki knowledge during a task and the actual
  result contradicted the wiki (e.g., API returned a different field than documented)
- **User correction** — User explicitly said the wiki is wrong about something
- **New session evidence** — After an ingest, a new page contradicts an existing one

Identify the affected page(s) from `wiki/index.md`. If multiple pages could apply,
read the index tags and pick the most specific match.

## Step 2 — Classify the change type

| Change type | Definition | Action |
|---|---|---|
| **Refinement** | New evidence adds nuance without invalidating prior claims | Update the page, add source, keep confidence |
| **Correction** | A claim was simply wrong | Fix the claim, document what changed and why |
| **Supersession** | The domain itself changed (API updated, new approach replaces old) | Create new page with `supersedes:`, update old with `superseded-by:` |
| **Contradiction** | New evidence conflicts, but old evidence also had basis | Add `## Contradictions` section, set confidence: medium, note both sides |

## Step 3 — Propose changes (confirm if user-triggered; proceed autonomously if Agent-triggered)

**User-triggered updates:** Present changes before applying:

```
Page: [[slug]]
Change type: correction

Current: "POST /calendar/v4/events accepts up to 100 items per batch"
Proposed: "POST /calendar/v4/events accepts up to 50 items per batch (rate limit
          was reduced in 2026-03)"
Reason: Current session returned 429 at item 51; confirmed by retrying with 50
Source: session-2026-04-14-feishu-batch

Apply? (yes / skip / modify)
```

**Agent-triggered updates during a task:** Apply immediately, then tell the user:
> "⚠ Wiki correction: [[slug]] claimed X, but this session shows Y. I've updated
> the page with the new evidence. Continuing with corrected understanding."

## Step 4 — Apply the change

1. Edit the page content (use the appropriate change type action from Step 2)
2. Update frontmatter:
   - `updated:` → today
   - Add current session ID to `sources:` array
   - Increment `session_count`
   - Adjust `confidence:` if threshold crossed (add corroboration → up; contradiction → down to medium)
3. For supersessions:
   - Old page: add `superseded-by: new-slug` + `## Superseded` section explaining what changed
   - New page: add `supersedes: [old-slug]`
4. For corrections on high-confidence pages: add a `## Change history` section noting what was wrong and when corrected (don't hide past mistakes — they're part of the knowledge)

## Step 5 — Downstream check

Search `wiki/pages/` for other pages that:
- Link to the updated page via `[[slug]]`
- Share the same primary tag
- Have `sources:` that include any of the same session IDs

For each downstream page, assess: does this change affect their claims?
- If yes → flag: "⚠ [[downstream-slug]] may be affected — it relies on the claim you just changed"
- Auto-update obvious formatting issues (update dates, broken links), but surface content
  changes to the user rather than auto-applying

## Step 6 — Contradiction sweep

Search `wiki/pages/` for all occurrences of the key term or API you just changed.
If any page now contradicts the updated understanding:
1. Add `## Contradictions` section: "This page conflicts with the updated [[changed-slug]]"
2. Set those claims to `confidence: medium`
3. Note the conflict without resolving it — contradictions are knowledge, not errors

## Step 7 — Update index and log

**wiki/index.md:** Update the one-line description if the page's scope or confidence changed.

**wiki/log.md:**
```
[YYYY-MM-DD HH:MM] update | <slug> | <change-type>: <brief reason> | source: <session-id>
```

## Step 8 — Report

Tell the user:
- Pages updated and change types applied
- Confidence changes (if any were promoted or demoted)
- Downstream pages flagged for review
- Contradictions found and left for resolution
