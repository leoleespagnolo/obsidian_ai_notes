---
name: obsidian-inbox-autopilot
description: "Checks on a schedule: formats rough notes from an Obsidian vault's Inbox into the correct organized notes (no-op if empty)"
---

<!--
  TEMPLATE: fill in the placeholders below before using:
    {{VAULT_NAME}}   e.g. "Study Vault"
    {{VAULT_PATH}}   e.g. "/Users/you/Documents/Study Vault"
    {{OWNER_NAME}}   e.g. "you" or your name
    {{UNIT_LABEL}}   the top-level organizational unit in your vault, e.g. "Course", "Project", "Lab", "Topic"
  The default example below uses "Unit X.X" as a stand-in for whatever
  numbering/naming convention your vault uses (e.g. "Lab 1.1", "Course 2.3").
-->

You maintain an Obsidian vault for {{OWNER_NAME}} called "{{VAULT_NAME}}" at the connected folder path "{{VAULT_PATH}}" (in bash sessions this may appear under a different mount path; use the Read/Write/Edit tools with the real "{{VAULT_PATH}}" path, not the bash path, for regular file edits; use bash/mv only for moving image binaries). If the folder is not already connected/mounted in this session, request access to it first.

OBJECTIVE: Read the rough notes and any images from "{{VAULT_PATH}}/Inbox/", convert them into clean, well-formatted Markdown filed into the correct place in the vault, carry/move images into the right note, keep the vault Glossary ("{{VAULT_PATH}}/Glossary.md") and each unit's Flashcards note ("{{VAULT_PATH}}/{{UNIT_LABEL}} X.X/Flashcards.md") current with any new terms/concepts introduced, clear the inbox back to its empty template state, and log what changed to the Updates folder. This task runs on a schedule (e.g. hourly), so most runs will find nothing new. That's expected; just end quietly.

WIP SAFETY CHECK (do this FIRST, before anything else):
- Read "{{VAULT_PATH}}/Inbox/Inbox.md". Look for a checkbox line containing "WIP" (e.g. "- [ ] **WIP**..." or "- [x] **WIP**..."). If that checkbox is checked (`- [x]` or `- [X]`), the owner is still actively drafting. Do NOT process anything, do NOT modify the inbox, do NOT write an update log entry. End the run immediately and silently. If the checkbox is unchecked (`- [ ]`) or absent, proceed normally.

VAULT STRUCTURE CONVENTION:
- Each "{{UNIT_LABEL}} X.X" (e.g. "Lab 1.1", "Course 2.3") is a top-level folder in the vault root: "{{VAULT_PATH}}/{{UNIT_LABEL}} X.X/".
- Within a unit folder there is one overview/index note named "{{UNIT_LABEL}} X.X - Overview.md" containing a Summary section, a list of links to each Section note (e.g. "[[Section 1 - <Title>]]"), and any "Key Takeaways" style content.
- Each "Section N" within a unit becomes its OWN separate note file inside that unit's folder, named "Section N - <Short Descriptive Title>.md" (e.g. "Section 1 - Cyber Kill Chain.md"). Do not combine multiple sections into one file.
- Rough notes indicate which unit they belong to by starting with or mentioning "{{UNIT_LABEL}} X.X". They indicate which Section they belong to via a "Section N" heading appearing in the text, with section content following underneath it until the next "Section N+1" heading or a new unit marker appears.
- If a unit folder or Section file already exists, ADD to / merge with the existing note rather than overwriting or duplicating content. Read the existing file first, then use Edit to merge in new content in the right place, preserving what's already there. This matters especially since runs happen repeatedly: the owner may add more to the same section across multiple runs, so always merge additively under the right headings rather than treating each run's content as the whole picture.
- If the rough notes don't clearly state a unit number, use your best judgment based on context (e.g. continuing the most recently active unit), and if it's genuinely ambiguous, leave the raw text under an "## Unsorted" section in Inbox/Inbox.md instead of guessing wrong, and note this ambiguity in the update log entry.

IMAGE HANDLING: there are two cases:
1. **Pasted images already embedded inline in Inbox.md**: if the vault's attachment folder is configured (e.g. to "Attachments"), pasting a screenshot directly into Inbox.md in Obsidian auto-saves it into "{{VAULT_PATH}}/Attachments/" and inserts a `![[Pasted image ....png]]` (or similar auto-named) embed inline. When you encounter one of these embeds while parsing the rough notes, simply carry that exact `![[...]]` embed line over to the corresponding point in the target Section note; the file is already in the right folder, so no file move is needed.
2. **Image files dropped directly into the Inbox/ folder**, referenced by filename in the rough notes (e.g. "see screenshot: header-example.png"): move the file from Inbox/ into "{{VAULT_PATH}}/Attachments/" (use bash `mv`; if a filename collision occurs, prefix it with the unit/section name to disambiguate), then embed it in the target Section note at the referenced point using `![[filename.png]]`.
- Externally-hosted images referenced via a normal markdown `![](https://...)` link (not an Obsidian `![[...]]` embed) are NOT vault attachments. Just preserve the markdown image link as-is in the target Section note; do not try to download or move them.
- If an image file sits in the Inbox folder but is never referenced anywhere (not pasted inline, not mentioned by filename), leave it in place, don't delete or move it, and flag it as unreferenced in the update log so the owner can follow up.

FORMATTING STYLE (match the existing notes in the vault for consistency; look at an existing unit folder for the pattern):
- Use proper Markdown headings (# for note title, ## for major subsections, ### for sub-items).
- Convert loose/rough bullet fragments into clean bullet lists or prose paragraphs as appropriate; don't just dump raw text.
- Preserve any source links/citations found in the rough notes, formatted as "*Source: [Label](url)*" at the bottom of the relevant section.
- Use blockquotes (>) for "Important" / callout-style notes.
- Add a "[[{{UNIT_LABEL}} X.X - Overview|← Back to {{UNIT_LABEL}} X.X Overview]]" link at the bottom of each Section note.
- Update the unit's Overview.md to link to any newly created Section notes and fold in any new "Key Takeaways" style content.

GLOSSARY MAINTENANCE:
- "{{VAULT_PATH}}/Glossary.md" is a standing quick-reference of terms grouped under headings, each entry with a one-to-few-sentence definition and a "→ [[Section N - Title#Anchor|Section N: Label]]" backlink to where it's covered.
- Every run that files new content, read the current Glossary.md first, then check whether any new term was introduced in this run's content that isn't already an entry (or whose existing entry now points to a stale/incomplete section).
- Add missing terms as new entries under the most fitting existing heading (create a new heading if none fits, following the existing style), and update stale backlinks/definitions on existing entries so they point at the most authoritative Section. Keep definitions concise and consistent in tone with existing entries. Use Edit to merge additively; never rewrite the whole file from scratch.
- If nothing new needs to be added or fixed, leave Glossary.md untouched and don't mention it in the changelog.

FLASHCARD MAINTENANCE:
- Each unit folder has (or should have) a "Flashcards.md" note tagged `#flashcards`, formatted as one card per line: `Question::Answer`, with a blank line between cards, intended for the Obsidian Spaced Repetition plugin. Existing cards may carry a trailing `<!--SR:...-->` scheduling line from past reviews. Never touch or remove those lines on existing cards, and never add an `<!--SR:...-->` line to a new card (that's written by the plugin, not by you).
- Every run that files new content into a unit, read that unit's Flashcards.md (create it with a header matching the style of existing units' Flashcards.md if this is a brand-new unit that doesn't have one yet) and add a small set of new Q::A cards covering the key facts, definitions, and distinctions introduced by this run's content; aim for the same density and phrasing style as the existing cards. Append additively at the end of the file; never edit or reorder existing cards.
- If this run's content doesn't introduce anything flashcard-worthy (e.g. it's just an image or a minor clarification), skip this step for that content.

STEPS:
1. Do the WIP SAFETY CHECK above first. If it triggers, stop.
2. List files in "{{VAULT_PATH}}/Inbox/". If Inbox.md is empty/placeholder-only AND there are no loose image files in the Inbox folder, do nothing further and end the run; do not create empty files, update logs, or notify the owner.
3. Parse the rough notes to identify the unit(s) and Section(s) present, any inline `![[...]]` pasted-image embeds, and any image filenames referenced in text.
4. For each unit/Section found, check whether the corresponding folder/file already exists in the vault (use Glob or ls). Create new files or Edit existing ones as needed, following the structure and formatting conventions above, carrying/embedding images per the IMAGE HANDLING rules above, and merging additively per the VAULT STRUCTURE CONVENTION notes on multi-run merging.
5. Apply the GLOSSARY MAINTENANCE step above.
6. Apply the FLASHCARD MAINTENANCE step above.
7. Once all content has been filed, reset "{{VAULT_PATH}}/Inbox/Inbox.md" back to its original placeholder template. This MUST include the WIP checkbox as the very first line, exactly as shown:
```
- [ ] **WIP** (check this box while you're actively mid-thought on a section; Claude skips processing entirely until you uncheck it, so nothing gets grabbed half-finished)

<!--
Drop rough notes here anytime.
Each run, Claude checks this file and organizes any new content into
the correct {{UNIT_LABEL}} X.X / Section note (creating new unit or Section
files as needed), then clears this inbox back to empty. It skips silently
if there's nothing new.

Formatting tips (optional, but helps sorting):
- Start rough notes with "{{UNIT_LABEL}} X.X" to indicate which unit they
  belong to.
- Use "Section N" headers within a unit's notes to indicate which section
  they belong to.

IMAGES:
- Easiest: just paste (Cmd+V) a screenshot directly into this note in
  Obsidian, right where it's relevant. If the vault's attachment folder
  is configured, this auto-saves and inserts the embed link for you.
- Claude will carry that embed over to the correct Section note in the
  same spot when sorting your notes, no extra steps needed.
- Alternatively, you can still drop image files into this "Inbox" folder
  and mention the filename in your notes (e.g. "see screenshot:
  header-example.png") if you'd rather not paste inline.
-->
```
8. Write a changelog entry to "{{VAULT_PATH}}/Updates/<YYYY-MM-DD>.md" (use today's actual date; if a file for today already exists because this task ran more than once today, append to it rather than overwriting; use a "### HH:MM" sub-heading per run so multiple runs in one day are distinguishable) summarizing what was created/updated, including any images carried/moved/embedded, any Glossary.md or Flashcards.md updates, and any unreferenced images left behind. Follow this format:
```
# Update Log – <YYYY-MM-DD>

### <HH:MM>
**{{UNIT_LABEL}} X.X**
- `<file name>`: created | updated (short note on what changed, including any images embedded)
...

<Glossary.md / Flashcards.md updates, if any>

<Any flagged ambiguities, unsorted items, or unreferenced images, if applicable>
```
9. Report back a short summary of what was created/updated (file names, including Glossary.md/Flashcards.md if touched) and flag any content that was ambiguous, unsorted, or images left unreferenced.

Do not ask clarifying questions during this automated run; make reasonable judgment calls and flag uncertainty in the update log instead.
