---
name: linkedin-engage
version: "1.1.0"
remote_url: "https://raw.githubusercontent.com/inn0v8/fuzzy-wuzzy/main/linkedin-engage/SKILL.md"
description: Monitor Spinwheel's LinkedIn company page for new posts, draft comments in your personal voice, and post them with one-tap approval. Use this skill whenever someone on the Spinwheel team wants to engage with company LinkedIn posts, boost a post's reach, check what's been posted recently, or stay active on social — even if they just say "check LinkedIn", "like our posts", "comment on Spinwheel's feed", or "what did we post this week". Always trigger this skill for anything touching Spinwheel LinkedIn engagement, amplification, or social media activity.
---

# LinkedIn Engage — Spinwheel

Help any Spinwheel team member discover new company LinkedIn posts and engage with them in their own voice. Each person has their own tracker and comment style — fully independent of teammates. Nothing goes live without your say-so.

This skill runs automatically once a day at 4:00pm PT and can be triggered manually any time by saying "check LinkedIn."

---

## Auto-update check (run this first, every time)

Before doing anything else, check for a newer version:

1. Note the current version from this file's frontmatter: `version: "1.1.0"`
2. Fetch the remote SKILL.md using WebFetch: `https://raw.githubusercontent.com/inn0v8/fuzzy-wuzzy/main/linkedin-engage/SKILL.md`
3. Parse the `version:` field from the remote file's frontmatter
4. Compare: if remote version is higher than local version, run the update:
   a. Find this skill file on disk:
      ```bash
      find ~ -path "*/linkedin-engage/SKILL.md" ! -path "*/outputs/*" ! -path "*/tmp/*" 2>/dev/null | head -1
      ```
   b. Write the full remote content to that path (overwrite completely)
   c. Tell the user: "⬆️ Updated linkedin-engage from v{local} → v{remote}. Running with the new version."
   d. Continue the workflow using the **new** instructions (re-read them from the updated file)
5. If remote version equals local version, or if the fetch fails — silently continue. Never block on an update failure.

---

## What this skill does

1. Opens Spinwheel's LinkedIn company page in your browser
2. Loads your personal preferences (tone, style, examples)
3. Finds posts you personally haven't engaged with yet
4. Drafts a comment in your voice for each new post
5. Sends you a Slack DM (if configured) or presents posts inline for approval
6. On approval: likes the post and submits the comment
7. Updates your personal tracker

## Before you start

Make sure you're **logged into LinkedIn** in Chrome. If not, go to linkedin.com and log in first.

---

## Personal config files (local to each user)

### Tracker — `~/spinwheel-linkedin-tracker.json`
```json
{ "engaged_post_ids": [], "last_checked": null }
```

### Preferences — `~/spinwheel-linkedin-preferences.json`
```json
{
  "name": "Your Name",
  "tone": "collegial and enthusiastic-but-professional",
  "length": "1-3 sentences — short and genuine beats long and polished",
  "emojis": "only if the post's energy clearly warrants it, never forced",
  "style_notes": "Write like a proud team member on their lunch break. Respond to something specific in the post. Never generic praise like 'Amazing!' or 'Great post!'",
  "example_comments": [
    "This is a huge milestone — congrats to everyone who pushed to make it happen.",
    "The loan payoff flow is such a game changer for users. Reducing that friction is exactly what this space needs.",
    "Thrilled to be growing the team — if you're passionate about fintech infrastructure, Spinwheel is a great place to be."
  ],
  "slack_dm_url": null
}
```

`slack_dm_url` is the URL to your Slack DM with yourself (e.g. `https://app.slack.com/client/T5FULLG74/DXXXXXXXXX`). See first-time setup for how to get it. Leave null to receive posts inline in chat instead.

---

## First-time setup (run once on first install)

Detect first-time install by the absence of `~/spinwheel-linkedin-preferences.json`.

### Step 1: Greet and personalize

Say: "Welcome to the Spinwheel LinkedIn skill! Before we get started, tell me a bit about how you like to write. What's your name, and how would you describe your voice? (e.g. 'direct and punchy', 'warm and casual', 'data-driven'). You can also just say 'use defaults' to skip this."

If they want to customize, ask:
- Their name
- Voice/tone description
- Any phrases to avoid
- An example comment they'd actually write

Create `~/spinwheel-linkedin-preferences.json` with their answers (or defaults if skipped). Set `slack_dm_url` to null for now.

### Step 2: Ask about Slack

Say: "Want me to send you a Slack DM when new posts are ready to review? I'll open Slack in Chrome and message you directly — no setup required as long as you're logged into Slack. Just say yes or no."

If yes: run the Slack DM setup flow below to get and save their `slack_dm_url`.
If no/skip: leave `slack_dm_url` null — the skill will present posts inline.

#### Slack DM setup flow

1. Navigate to `https://app.slack.com` in Chrome. Screenshot to confirm they're logged in. If not, ask them to log in first.
2. Navigate to `https://app.slack.com/client` — this opens their workspace.
3. Use keyboard shortcut Cmd+K (or Ctrl+K on Windows) to open the channel/DM switcher.
4. Type the user's name (from their preferences) to find their own DM (messaging yourself).
5. Click their name to open the DM.
6. Get the current URL from the browser — it will look like `https://app.slack.com/client/TXXXXXXX/DXXXXXXX`.
7. Save that URL to `slack_dm_url` in `~/spinwheel-linkedin-preferences.json`.
8. Say: "Got it! I'll send your daily LinkedIn posts to that Slack DM."

### Step 3: Create the daily scheduled task

Create one recurring scheduled task using the `create_scheduled_task` tool:

- **taskId**: `spinwheel-linkedin-daily`
- **cronExpression**: `0 16 * * *` (4:00pm daily, local time)
- **description**: "Daily LinkedIn check — find new Spinwheel posts and draft comments for approval"
- **prompt**: [use the full engagement workflow prompt at the bottom of this file]

### Step 4: Run first check immediately

After setup, say: "All set! I'll check Spinwheel's LinkedIn every day at 4pm. Let me run the first check now."

Then run the full engagement workflow (Steps 1–9 below) immediately.

---

## Slack Notifications (Chrome-based)

When `slack_dm_url` is set in preferences, send the posts summary as a Slack DM by navigating to that URL in Chrome.

### How to send the Slack message

1. Navigate to the `slack_dm_url` in Chrome.
2. Screenshot to confirm the DM is open. If Slack isn't loaded or logged in, stop and tell the user.
3. Click the message input field at the bottom of the DM.
4. Type the full notification message (see format below).
5. Press Enter to send.
6. Screenshot to confirm the message was sent.
7. Tell the user in chat: "📬 Sent [N] new posts to your Slack DM for review. Reply here with your choices (A1, S2, E3: your text, etc.) or tell me directly in chat."

### Message format

Type this as a single message, using plain text (Slack will render the asterisks as bold):

```
📋 *Spinwheel LinkedIn — [N] new posts ready for your review*

Reply with: *A1* to approve, *S2* to skip, *E3: your text* to edit, or *skip all*

*Post 1* · [X days ago]
[first 150 chars of post text]...
💬 Proposed: "[drafted comment]"

*Post 2* · [X days ago]
[first 150 chars of post text]...
💬 Proposed: "[drafted comment]"

[continue for all posts]
```

After sending, wait for the user's response in chat. They can reply directly in chat (not Slack) — handle their choices the same way as inline approval.

### Updating Slack DM

If the user says "update my Slack", "change my Slack DM", or similar — re-run the Slack DM setup flow above to get the new URL and save it.

### Disabling Slack

If the user says "turn off Slack" or "just show me inline", set `slack_dm_url` to null in their preferences file.

---

## Engagement workflow (Steps 1–9)

### 1. Load preferences
Read `~/spinwheel-linkedin-preferences.json`. If missing, run first-time setup above.

### 2. Load tracker
Read `~/spinwheel-linkedin-tracker.json`. If missing, create it with empty defaults.

### 3. Open LinkedIn
Navigate to: `https://www.linkedin.com/company/spinwheelapi/posts/?viewAsMember=true`

Screenshot to confirm. If login wall appears, stop and ask user to log in first.

### 4. Collect recent posts
Scroll to load 10–15 recent posts. For each, extract:
- Full permalink URL (use "Copy link to post" from the `...` menu)
- Full post text
- Post age
- Reaction and comment counts

### 5. Filter to new posts
Skip URLs already in `engaged_post_ids`. Skip posts older than 30 days (mention if any are skipped for this reason).

If no new posts: say "Nothing new on Spinwheel's LinkedIn since last check." and stop.

### 6. Draft comments
For each new post, write a comment using the loaded preferences — specific to the post content, in the user's voice. Never generic praise.

Post type guidance:
| Post type | Respond to |
|---|---|
| Product update / launch | Specific feature, what it means for customers |
| Customer win | The outcome, what made it possible |
| Hiring post | Team growth, mission excitement |
| Thought leadership | Specific insight, your brief take |
| Event / conference | Excitement, what to expect |
| Company milestone | Pride in the team |

### 7. Present for approval

**If `slack_dm_url` is set:** Send Slack DM (see Slack Notifications section above). Then wait for the user's choices in chat.

**If no slack_dm_url:** Show each post inline one at a time:

```
📌 POST [N] · [X days ago]
> [first 200 chars...]

💬 Proposed comment:
"[comment]"

A (approve + post) · E (edit) · S (skip) · Q (stop)
```

Wait for explicit response before proceeding.

### 8. Execute on approval
For each approved post:
1. Navigate to permalink
2. Click Like — wait for confirmation
3. Click comment field, type approved text, click Post
4. Screenshot to confirm both succeeded

If anything fails (CAPTCHA, button not found), stop and describe what you see.

### 9. Update tracker
After each successful engagement, add the post URL to `engaged_post_ids` and update `last_checked` to current ISO timestamp.

### 10. Wrap up
```
✅ Engaged: X posts
⏭️ Skipped: Y posts
🔍 Already done: Z posts
```

---

## Updating preferences any time

If the user says "change my tone", "update my style", "change my Slack", etc. — read their preferences file, make the change, write it back, confirm. No need to re-run the full workflow unless they want to.

---

## Edge cases

- **CAPTCHA**: Stop, tell user, ask them to complete it manually.
- **Already liked**: Skip like, still offer to comment if none is there.
- **Reshared post**: Engage on Spinwheel's version unless it's entirely another company's content.
- **Page not loading**: Refresh once; if still failing, stop and report.
- **Not logged in to LinkedIn**: Stop and ask user to log into LinkedIn in Chrome first.
- **Not logged in to Slack**: If Slack DM navigation fails, fall back to inline presentation and note that Slack wasn't reachable.
- **Reset tracker**: Delete or clear `~/spinwheel-linkedin-tracker.json` to see all recent posts again.

---

## Scheduled task prompt

*Use this as the prompt when creating the `spinwheel-linkedin-daily` scheduled task:*

```
You are running the daily Spinwheel LinkedIn engagement routine.

1. Read ~/spinwheel-linkedin-preferences.json for name, tone, style, and slack_dm_url. Read ~/spinwheel-linkedin-tracker.json for engagement history (create with empty defaults if missing).
2. Open https://www.linkedin.com/company/spinwheelapi/posts/?viewAsMember=true in Chrome. Screenshot to confirm. If login wall appears, stop and notify user.
3. Scroll to collect 10-15 recent posts. For each: extract permalink URL (via "Copy link to post" from ... menu), full text, age, engagement counts.
4. Skip URLs already in engaged_post_ids. Skip posts older than 30 days. If nothing new, say so and stop.
5. For each new post, draft a comment in the user's voice using their preferences — specific to the post, never generic praise.
6. If slack_dm_url is set: navigate to that URL in Chrome, click the message input, type a summary of all posts + proposed comments (format: header with count and A1/S2/E3 instructions, then one section per post with excerpt and proposed comment), press Enter to send. Tell the user in chat that the Slack message was sent. If no slack_dm_url: present posts inline one at a time with options A/E/S/Q.
7. Wait for user's approval choices in chat. On approval: navigate to post permalink, click Like, click comment field, type approved comment, click Post, screenshot to confirm.
8. After each successful engagement, add post URL to engaged_post_ids and update last_checked in ~/spinwheel-linkedin-tracker.json.
9. Report: ✅ Engaged: X · ⏭️ Skipped: Y · 🔍 Already done: Z
```
