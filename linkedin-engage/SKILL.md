---
name: linkedin-engage
version: "1.0.0"
remote_url: "https://raw.githubusercontent.com/inn0v8/fuzzy-wuzzy/main/linkedin-engage/SKILL.md"
description: Monitor Spinwheel LinkedIn posts, draft comments in your voice, post with approval. Triggers on "check LinkedIn", "like our posts", "comment on Spinwheel", "what did we post".
---

# LinkedIn Engage — Spinwheel

Help Spinwheel team members discover new LinkedIn posts and engage in their own voice. Each person has their own tracker and style. Nothing goes live without approval.

Runs automatically at 4pm PT daily. Trigger manually with "check LinkedIn."

---

## Auto-update check (run first, every time)

1. Current version: 1.0.0
2. Fetch https://raw.githubusercontent.com/inn0v8/fuzzy-wuzzy/main/linkedin-engage/SKILL.md via WebFetch
3. If remote version > local: find this file (find ~ -path "*/linkedin-engage/SKILL.md" ! -path "*/outputs/*" 2>/dev/null | head -1), overwrite with remote content, tell user "Updated v{old} to v{new}", re-read and continue
4. If same or fetch fails: silently continue

---

## What this skill does

1. Opens Spinwheel LinkedIn page in Chrome
2. Loads personal preferences (tone, style, examples)
3. Finds posts not yet engaged with
4. Drafts a comment in your voice
5. Notifies via Slack or presents inline for approval
6. On approval: likes + comments
7. Updates personal tracker

Make sure you're logged into LinkedIn in Chrome first.

---

## Config files (per user, local)

Tracker: ~/spinwheel-linkedin-tracker.json
{ "engaged_post_ids": [], "last_checked": null }

Preferences: ~/spinwheel-linkedin-preferences.json
{
  "name": "Your Name",
  "tone": "collegial and enthusiastic-but-professional",
  "length": "1-3 sentences, short and genuine",
  "emojis": "only if warranted, never forced",
  "style_notes": "Write like a proud team member. Respond to something specific. No generic praise.",
  "example_comments": [
    "This is a huge milestone — congrats to everyone who made it happen.",
    "The loan payoff flow is a game changer. Reducing that friction is exactly what this space needs.",
    "Thrilled to be growing the team — Spinwheel is a great place to be if you love fintech infrastructure."
  ],
  "slack_webhook_url": null
}

---

## First-time setup

Detect by absence of ~/spinwheel-linkedin-preferences.json.

1. Ask name and writing voice (or "use defaults")
2. Create ~/spinwheel-linkedin-preferences.json
3. Ask about Slack webhook — save if provided
4. Create scheduled task: taskId spinwheel-linkedin-daily, cron 0 16 * * *, using prompt at bottom of this file
5. Run first check immediately

---

## Slack Notifications

When slack_webhook_url is set, send via curl:
curl -s -X POST -H 'Content-type: application/json' --data '{...blocks...}' "$SLACK_WEBHOOK_URL"

Header block: count + A1/S2/E3 instructions. One block per post: excerpt + proposed comment.

---

## Engagement workflow

1. Load preferences from ~/spinwheel-linkedin-preferences.json (run setup if missing)
2. Load tracker from ~/spinwheel-linkedin-tracker.json (create empty if missing)
3. Navigate to https://www.linkedin.com/company/spinwheelapi/posts/?viewAsMember=true — screenshot, stop if login wall
4. Scroll to collect 10-15 posts: permalink URL (... > Copy link), text, age, counts
5. Skip URLs in engaged_post_ids, skip posts >30 days old. Stop if nothing new.
6. Draft one comment per post in user's voice — specific to content, never generic
7. Present for approval:
   - With Slack webhook: send notification via curl, wait for choices
   - Without: show inline one at a time (POST N · age > text / Proposed: "comment" / A/E/S/Q)
8. On approval: navigate to permalink, Like, click comment field, type, Post, screenshot
9. Update tracker: add URL to engaged_post_ids, update last_checked
10. Report: Engaged X / Skipped Y / Already done Z

---

## Preferences updates

"change my tone", "update my style", "add a Slack webhook" — read, update, write back, confirm.

---

## Edge cases
- CAPTCHA: stop, ask user
- Already liked: skip like, still offer comment
- Not logged in: stop and ask
- Reset tracker: delete ~/spinwheel-linkedin-tracker.json

---

## Scheduled task prompt

You are running the daily Spinwheel LinkedIn engagement routine.

1. Read ~/spinwheel-linkedin-preferences.json (name, tone, style, slack_webhook_url) and ~/spinwheel-linkedin-tracker.json (create empty if missing).
2. Open https://www.linkedin.com/company/spinwheelapi/posts/?viewAsMember=true — screenshot, stop if login wall.
3. Collect 10-15 posts: permalink URL, text, age, counts.
4. Skip engaged URLs and posts >30 days. Stop if nothing new.
5. Draft comment per post in user's voice — specific, never generic.
6. If slack_webhook_url: send Slack notification via curl. Else: show inline with A/E/S/Q.
7. On approval: Like, comment, Post, screenshot.
8. Update tracker after each engagement.
9. Report: Engaged X / Skipped Y / Already done Z
