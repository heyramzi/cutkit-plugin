---
name: cutkit-teleprompter
description: Writes and manages the scripts in a person's CutKit iPhone teleprompter over the CutKit MCP tools. Use when someone asks to draft a video script they'll read on camera, put a script on their teleprompter, start one from a hook template or story beats, check or tighten a script, plan a month of scripts, or list, label, pin, archive or delete their CutKit scripts.
---

# CutKit Teleprompter

CutKit is an iPhone teleprompter. The script scrolls as the person speaks, and the app records the take. All of a person's scripts live in one library: what you write with these tools shows up on the phone the next time CutKit opens, and edits made on the phone show up here.

The `cutkit` MCP server acts for one library, the account the person signed in with. There are 18 tools.

| Job | Tools |
| --- | --- |
| Read the library | `list_scripts`, `get_script` |
| Write and change scripts | `create_script`, `update_script`, `delete_script` |
| Start from a shape | `list_templates`, `get_template`, `start_from_template`, `start_story`, `plan_month` |
| Grade a script | `lint_script`, `check_script` |
| Labels | `list_labels`, `rename_label`, `delete_label`, `set_label_color` |
| Who the videos are for | `get_profile`, `set_profile` |

## Traps

- `list_scripts` leaves the bodies out. Call `get_script` (by `id` or `title`) before you change a script's words.
- `update_script` takes an id. Look it up by title with `list_scripts` first. Only the fields you send change, and `labels` replaces the whole list.
- Unless the person says delete, archive instead. `update_script` with `status: "archived"` hides it from the library and keeps it. `status` is one of `draft`, `to_record`, `recorded`, `archived`.
- `check_script` has a quota of 100 a day per library, and every answer says how many are left. The same text checked again within 24 hours is free. `lint_script` costs nothing, so lint until clean, then spend a check.
- `plan_month` needs CutKit Pro or a trial on the account. It adds 30 scripts at once, so confirm the topics before you call it.
- `rename_label` onto a label that exists merges the two. Say so before you do it.

## Write for the teleprompter

The phone listens and matches each spoken word against the screen, so the body holds only words the person will say.

- No `[pause]`, `(smile)` or `B-roll: desk shot`. The app waits for the person to say them and the scroll stalls. Put notes like that in the chat.
- No markdown: no bold, links or tables. Plain sentences are safest.
- Keep sentences short, mostly 6 to 15 words. A long one wraps 4 times on a phone and the reader loses their place.
- One paragraph per beat, with a blank line between, so there's a place to breathe and restart a take.
- The app counts about 140 words a minute, so a 30 second read runs near 70 words.
- The title isn't read on camera. Keep it short and findable, like "Launch video intro". With no title, the first line becomes one.

## Read the profile first

Call `get_profile` before you write or check. `audience` is who the person makes videos for, and once it's set, `check_script` adds `audience_fit`, which fails a script that talks to somebody else. If `audience` is empty, ask who the videos are for, write the 3 fields (`about`, `audience`, `offer`, each under 1,200 characters), show them, then `set_profile`. Never invent a credential for `about`.

## Workflows

The quote under each heading is what the person might say.

### Draft a script and put it on the phone

"Write a one minute intro for my cold brew video."

1. Ask only what you can't guess, like their name or channel.
2. Draft about 140 words: a hook, who they are, what the video covers, why stay.
3. `lint_script` with the `body`. Fix what it flags.
4. `create_script` with `title` and `body`. Tell the person the title and word count from the answer, and that it's on the phone next time CutKit opens.

### Start from a hook template

"Start a script from a question hook about pricing."

1. `list_templates`, with `family` to narrow it (`promise`, `diagnosis`, `belief-break`, `in-medias-res`, `tribe`, `artefact`, `number`, `question`, `news`, `contradiction`, `warning`).
2. `get_template` on the one that fits, to see its examples and the way it goes wrong.
3. `start_from_template` with `templateId` and a `title`. The new script has the opening line and re-hook in place and a `[bracket]` for every beat.
4. Write real words over every bracket, then `update_script`. A bracket left in fails the check.

For a story, `start_story` with a `topic` gives the beats (stakes, the big question, the head fake, a re-hook) as brackets to fill.

### Check and tighten a script

"Check my Launch video intro and fix it."

1. `list_scripts`, then `get_script` to read it.
2. `lint_script` with the `id`, fix, save with `update_script`.
3. `check_script` with the `id`. It starts with a grade from A+ to F and the score out of 100, then each failing check with its fix. A failing check can name `templateIds`; read one with `get_template` and use its shape.
4. Apply the fixes and save, then check again. You're done when nothing the person cares about still fails, or when a fix would mean inventing facts. Report the grade and what still fails. The grade is about craft. Never say it predicts views.

To cut a script to a length, count at 140 words a minute, cut whole sentences before trimming words, and save with `update_script`.

### Plan a month

"Give me a month of scripts on pricing, hiring and tools."

1. Confirm 1 to 6 topics.
2. `plan_month` with `topics`. It adds 30 drafts titled Day 1 to Day 30, each labelled with its topic.
3. Offer to fill the first few with the template workflow above.

### Organize the library

- Pin: `update_script` with `pinned: true`. Move along the pipeline with `status: "to_record"` or `"recorded"`.
- Group: `update_script` with `labels`, then `list_labels` to see counts.
- Color: `set_label_color` with one of `red`, `orange`, `amber`, `green`, `teal`, `sky`, `blue`, `violet`, `pink`, or `null` to clear.
- Clean up: `rename_label` to fix a name, `delete_label` to drop a label (the scripts stay).

## Errors

| Message | What to do |
| --- | --- |
| `invalid_token` | The sign-in expired. Ask the person to run `/mcp` and authenticate `cutkit` again. |
| `not_found` | No live script with that id. Call `list_scripts` and use a current one. |
| `title_too_long`, `body_too_long` | Keep the title under 200 characters, or split the body into 2 scripts. |
| `library_full` | 2,000 live scripts. Ask which to delete or archive. |

Read only the scripts the person points you to, and ask before you delete one.

Help: getcutkit.com/support or hello@getcutkit.com.
