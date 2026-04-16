---
name: linkedin-engage
description: Monitor Spinwheel's LinkedIn company page and the founder's personal profile, draft comments in the user's voice, and post them only after explicit approval. Trigger on requests to engage with, amplify, comment on, or check Spinwheel LinkedIn activity.
metadata:
  version: "1.3.0"
---

# LinkedIn Engage — Spinwheel

Help a Spinwheel team member discover new posts from the Spinwheel company page and the founder's personal LinkedIn profile, then engage with them in their own voice. Each user has their own tracker and comment style. Nothing goes live without explicit approval.

Runs daily at 4:00pm local time (via cron) and can be triggered manually by saying "check LinkedIn".

---

## Setup: resolve the data directory

Run this first on every invocation. It uses the Claude Code project dir env var, with a fallback to the current working directory:

```bash
WORKSPACE="${CLAUDE_PROJECT_DIR:-$(pwd)}"
# If cwd is a parent of the workspace, walk down one level
if [ ! -d "$WORKSPACE/linkedin-engage" ] && [ -d "$WORKSPACE/LinkedIn_Routine/linkedin-engage" ]; then
    WORKSPACE="$WORKSPACE/LinkedIn_Routine"
fi
DATA_DIR="$WORKSPACE/.claude/linkedin-data"
mkdir -p "$DATA_DIR"
echo "Data directory: $DATA_DIR"
```

If `$WORKSPACE/linkedin-engage/SKILL.md` does not exist after resolution, stop and ask the user which directory is their workspace.

---

## Personal config files

All config files live in `$DATA_DIR`.

### Tracker — `$DATA_DIR/tracker.json`
```json
{ "engaged_post_ids": [], "last_checked": null }
```

### Preferences — `$DATA_DIR/preferences.json`
```json
{
  "name": "Your Name",
  "tone": "collegial and enthusiastic-but-professional",
  "length": "1-2 sentences — short and genuine beats long and polished",
  "emojis": "only if the post's energy clearly warrants it, never forced",
  "style_notes": "Write like a proud team member on their lunch break. Respond to something specific in the post. Never generic praise like 'Amazing!' or 'Great post!'.",
  "example_comments": [
    "This is a huge milestone — congrats to everyone who pushed to make it happen.",
    "The loan payoff flow is such a game changer for users. Reducing that friction is exactly what this space needs.",
    "Thrilled to be growing the team — if you're passionate about fintech infrastructure, Spinwheel is a great place to be."
  ],
  "slack_dm_url": null,
  "slack_method": null
}
```

- `slack_method`: `"mcp"` (Slack MCP), `"chrome"` (browser fallback), or `null` (inline in chat).
- `slack_dm_url`: only needed for Chrome fallback.

---

## First-time setup

After establishing `DATA_DIR`, check whether `$DATA_DIR/preferences.json` exists. If not, this is a first-time install.

**Migration:** if `$DATA_DIR/preferences.json` is missing but a `spinwheel-linkedin-preferences.json` file exists elsewhere in the workspace, copy it over and skip setup.

### Step 1: personalize

Ask the user for name, voice/tone, phrases to avoid, and one example comment they'd actually write. Or accept "use defaults".

Write answers to `$DATA_DIR/preferences.json` with `slack_dm_url: null` for now.

### Step 2: Slack

Ask: "Want me to DM you in Slack when new posts are ready? Options: Slack MCP (cleanest), Chrome fallback, or skip (inline)."

**If Slack MCP:**
1. Check the current tool list for any Slack MCP tool (tool names starting with `mcp__claude_ai_Slack__` beyond `authenticate` / `complete_authentication`).
2. If only the auth tools are available, call `mcp__claude_ai_Slack__authenticate`, follow the returned flow, then call `mcp__claude_ai_Slack__complete_authentication`. After auth, re-check the tool list for send-message tools.
3. Send a test DM to the user. If it lands, save `slack_method: "mcp"` and `slack_dm_url: null`.
4. If MCP is unavailable or fails after auth, offer Chrome fallback.

**If Chrome fallback:** run the Chrome Slack DM flow below; save `slack_method: "chrome"` and the DM URL.

**If skip:** save `slack_method: null`.

#### Chrome Slack DM setup

1. Navigate to `https://app.slack.com/client` in Chrome. Screenshot to confirm logged in.
2. Press Cmd+K to open the switcher; type the user's name; click their own DM.
3. Copy the URL (`https://app.slack.com/client/TXXXXXXX/DXXXXXXX`) and save to `slack_dm_url`.

### Step 3: schedule the daily task

Use the `CronCreate` tool (load its schema via `ToolSearch` with `select:CronCreate` if not yet available) to create the recurring task:

- **schedule**: `0 16 * * *` (4:00pm local time)
- **name/id**: `spinwheel-linkedin-daily`
- **prompt**: the "Scheduled task prompt" at the bottom of this file

Note to the user that local time on the executing host is used — if they need Pacific time specifically, confirm the host timezone.

### Step 4: run first check

Say: "All set — running the first check now." Then execute the engagement workflow below.

---

## Slack notifications

When `slack_method` is set, send posts as a Slack DM before asking for approval inline.

### Message format

```
📋 *Spinwheel LinkedIn — [N] new posts ready for review*

Reply: *A1* to approve, *S2* to skip, *E3: your text* to edit, or *skip all*

*Post 1* · [Source] · [age]
[first 150 chars]...
💬 Proposed: "[drafted comment]"
```

After sending, tell the user in chat: "📬 Sent [N] new posts to your Slack DM. Reply here with your choices."

### Method dispatch

- **`mcp`**: call the Slack MCP send-message tool (name starts with `mcp__claude_ai_Slack__`). On failure, fall back to inline and tell the user.
- **`chrome`**: navigate to `slack_dm_url`, screenshot to confirm, click the input, type, press Enter, screenshot again. On failure, fall back to inline.
- **`null`**: show posts inline one at a time with options A / E / S / Q.

---

## Engagement workflow

### 1. Load state
Read `$DATA_DIR/preferences.json` and `$DATA_DIR/tracker.json` (create tracker with empty defaults if missing). If preferences is missing, run first-time setup.

### 2. Collect Spinwheel posts
Navigate to `https://www.linkedin.com/company/spinwheelapi/posts/?viewAsMember=true`. Screenshot. If a login wall appears, stop and ask the user to log in.

Scroll to load ~5 recent posts. For each, extract: permalink (via "Copy link to post" from `...` menu), full text, age, reaction/comment counts. Source label: `Spinwheel`.

### 3. Collect founder posts
Navigate to `https://www.linkedin.com/in/theinnovativeone/recent-activity/all/`. Same extraction. Source label: `Tomás`. If the page fails, skip and note in the summary.

### 4. Filter
Merge all posts. Drop URLs already in `engaged_post_ids`. Drop posts older than 7 days (note any dropped for age). If nothing new remains, say so and stop.

### 5. Draft comments
Write a comment for each new post using the loaded preferences — specific to the post content, in the user's voice. Never generic praise.

- **Spinwheel company posts**: proud team member amplifying company news.
- **Tomás's posts**: supportive colleague engaging with his idea.

| Post type | Respond to |
|---|---|
| Product update / launch | Specific feature, customer impact |
| Customer win | Outcome, what made it possible |
| Hiring post | Team growth, mission |
| Thought leadership | Specific insight, brief take |
| Event / conference | Excitement, what to expect |
| Company milestone | Pride in the team |
| Founder's personal take | Engage with the idea, add a perspective |

### 6. Present for approval
Dispatch via the method in preferences (see Slack notifications above). Wait for explicit approval before posting anything.

### 7. Execute
For each approved post:
1. Navigate to permalink.
2. Click Like; wait for confirmation.
3. Click comment field, type approved text, click Post.
4. Screenshot to confirm both actions.

If anything fails (CAPTCHA, button not found), stop and describe what you see.

### 8. Update tracker
After each successful engagement, append the URL to `engaged_post_ids` and update `last_checked` to the current ISO timestamp.

### 9. Wrap up
```
✅ Engaged: X (Y Spinwheel, Z Tomás)
⏭️ Skipped: N
🔍 Already done: M
```

---

## Managing preferences

- **"Change my tone"** / **"update my style"** — read, update, write back.
- **"Change my Slack"** — re-run Slack setup (Step 2 above).
- **"Turn off Slack"** — set `slack_method: null`.
- **"Reset tracker"** — delete or clear `$DATA_DIR/tracker.json`.

## Updating this skill

The skill is versioned in the `metadata.version` frontmatter field. There is no automatic updater — updates are opt-in.

If the user asks to "update the linkedin-engage skill" or "check for a newer version":

1. Confirm the source repo with the user (e.g. `https://github.com/inn0v8/fuzzy-wuzzy`) — never fetch from an unverified URL without asking.
2. Fetch the remote SKILL.md and show the user a diff of the changes before writing.
3. Only overwrite after explicit approval.

Never fetch, execute, or overwrite the skill file without user consent.

---

## Edge cases

- **CAPTCHA** — stop, tell the user, ask them to complete it manually.
- **Already liked** — skip like; still offer to comment if none from the user is there.
- **Reshared post** — engage on Spinwheel's (or Tomás's) version unless it's entirely another company's content.
- **Page not loading** — refresh once; if still failing, skip that source and note it.
- **Not logged in to LinkedIn** — stop and ask the user to log in.
- **Not logged in to Slack** — fall back to inline and note that Slack wasn't reachable.

---

## Scheduled task prompt

Use this as the `prompt` when creating the `spinwheel-linkedin-daily` cron via `CronCreate`:

```
You are running the daily Spinwheel LinkedIn engagement routine.

1. Resolve data directory:
     WORKSPACE="${CLAUDE_PROJECT_DIR:-$(pwd)}"
     if [ ! -d "$WORKSPACE/linkedin-engage" ] && [ -d "$WORKSPACE/LinkedIn_Routine/linkedin-engage" ]; then
         WORKSPACE="$WORKSPACE/LinkedIn_Routine"
     fi
     DATA_DIR="$WORKSPACE/.claude/linkedin-data"
     mkdir -p "$DATA_DIR"
   Read $DATA_DIR/preferences.json (name, tone, style, slack_method, slack_dm_url) and $DATA_DIR/tracker.json (create with empty defaults if missing).
2. Open https://www.linkedin.com/company/spinwheelapi/posts/?viewAsMember=true in Chrome. Screenshot. If login wall, stop and notify the user. Scroll to collect up to 5 recent posts — for each: permalink (via "Copy link to post"), full text, age, engagement counts, source "Spinwheel".
3. Navigate to https://www.linkedin.com/in/theinnovativeone/recent-activity/all/ and do the same, labeled "Tomás". If the page fails, skip and note it.
4. Merge. Skip URLs already in engaged_post_ids. Skip posts older than 7 days. If nothing new, say so and stop.
5. Draft a comment for each new post in the user's voice — specific, never generic. Spinwheel posts: proud team member. Tomás's posts: supportive colleague.
6. Dispatch based on slack_method. "mcp": use a Slack MCP send-message tool (name starts with mcp__claude_ai_Slack__). "chrome": navigate to slack_dm_url, type, send. null: present inline one-at-a-time with A/E/S/Q. On failure, fall back to inline.
7. Wait for approval. On approval: navigate to permalink, Like, comment, screenshot to confirm.
8. Append each engaged URL to engaged_post_ids and update last_checked in $DATA_DIR/tracker.json.
9. Report: ✅ Engaged: X (Y Spinwheel, Z Tomás) · ⏭️ Skipped: N · 🔍 Already done: M
```
