---
name: sage-update
description: >
  Revise existing wiki pages when knowledge changes, new sources contradict prior
  understanding, or a claim needs correction. Use this skill when the user has
  new information that updates or contradicts something already in the wiki,
  wants to correct a factual error, or has a new source that supersedes old content.
  Trigger on: "update my wiki", "correct wiki page", "revise wiki", "wiki update",
  "sage update", "the wiki is wrong about", "supersede this", "this contradicts
  my wiki", "update the page about". Treat any user correction of a wiki claim
  as a trigger for this skill.
---

# sage-update — Revise Wiki Pages

Updates one or more existing pages with new information, corrections, or
supersessions. Every change requires a source — this is what keeps the wiki
auditable and trustworthy over time.

## Step 0 — Find the wiki

Search upward from the current directory for `SCHEMA.md`. Fallback: `~/wikis/`.
Read it fully before proceeding.

## Step 1 — Identify what needs to change

The user will either:
- Point to a specific page: "update `[[transformer-attention]]` with this new paper"
- Describe a claim to correct: "the wiki says attention is O(n²) but sparse attention changes that"
- Provide a new source that changes prior understanding

If the target page isn't clear, read `wiki/index.md` and identify the most relevant
slug(s). Confirm with the user if multiple pages could apply.

## Step 2 — Read the current page(s)

Read every page you plan to modify. Don't propose changes based on the index alone.

Also read the new source in full if provided.

## Step 3 — Propose changes (WAIT for confirmation)

For each affected page, present changes in this format before writing anything:

---

**Page:** `[[slug]]`

| | Content |
|---|---|
| **Current** | _exact quote from the existing page_ |
| **Proposed** | _replacement text_ |
| **Reason** | _why this change is warranted_ |
| **Source** | _URL, file path, or description — required_ |

Proceed? (yes / skip / modify)

---

Present all proposed changes up front, then wait for the user to approve each one.
Never apply changes in batch without per-change confirmation.

If the user says "just do it" or "apply all", confirm once: "Apply all N changes?" —
but don't silently assume blanket approval.

## Step 4 — Apply approved changes

For each approved change:
1. Edit the page content
2. Update the `updated` date in frontmatter
3. Update `confidence` if the new source changes the reliability of claims
4. If the change is a **supersession** (new info makes old understanding invalid):
   - Set `superseded-by: new-slug` on the old page (or the updated page itself)
   - Add a `## Superseded Claims` section noting what changed and why

## Step 5 — Downstream check

After applying changes, search for other pages that reference the updated slug via
`[[slug]]` or share its tags. For each:
- Does the change affect their claims?
- If yes, flag it: "[[downstream-page]] may need updating — it relies on the
  claim you just changed."

Don't auto-update downstream pages. Surface them for the user to decide.

## Step 6 — Contradiction sweep

Search the wiki for all occurrences of the key term or claim you just changed.
If any page now contradicts the updated understanding:
- Add a `## Contradictions` section to the contradicting page
- Note the conflict and link to the updated page
- Set those claims to `confidence: medium` until resolved

## Step 7 — Update index and log

**wiki/index.md:** Update the one-line description for any page whose title or
scope changed significantly.

**wiki/log.md:**
```
[YYYY-MM-DD HH:MM] update | <slug> | <brief reason> | source: <source>
```

## Step 8 — Report

Tell the user:
- Pages updated: list
- Claims changed: brief summary of what shifted
- Downstream pages that may need review: list with reason
- Contradictions flagged: if any
