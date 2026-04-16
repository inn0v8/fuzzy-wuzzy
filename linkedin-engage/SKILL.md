---
name: linkedin-engage
description: Monitor Spinwheel's LinkedIn company page AND the founder's personal profile for new posts, draft comments in your personal voice, and post them with one-tap approval. Use this skill whenever someone on the Spinwheel team wants to engage with company or founder LinkedIn posts, boost a post's reach, check what's been posted recently, or stay active on social — even if they just say "check LinkedIn", "like our posts", "comment on Spinwheel's feed", "engage with Tomas's posts", or "what did we post this week". Always trigger this skill for anything touching Spinwheel LinkedIn engagement, amplification, or social media activity.
metadata:
  version: "1.2.7"
  remote_url: "https://raw.githubusercontent.com/inn0v8/fuzzy-wuzzy/main/linkedin-engage/SKILL.md"
---

                    ░▒▒▓▓▓▓▓▒▒░░░▒▒▓▓▓████████▓▓▓▒▒░                  
              ░▒▓█████▓▒░░▒▓▓████████████████████████▓▒░             
           ▒▓███████▒░░▓█████████████████████████████████▓▒          
        ░▓███████▓░░▒███████████████████████████████████████▓░       
      ░▓████████░░▒█████████████▓▒░░  ░▒▒▒▒░░░░▒▒▓████████████▓░     
     ▒████████▒ ▒███████████▓▒░         ▒▓█████▓▓▒░░▓███████████▓    
   ░▓████████▒ ▓██████████▒                ▒███████▓▒░▒▓██████████░  
   █████████▒ ▓█████████▓                    ▓████████▒ ▒██████████░ 
  ▓████████▓ ▒█████████▒                      ▒████████▓░░█████████▓ 
 ▒█████████░░█████████▒                        ▒█████████ ░█████████▒
 ▓████████▓ ▒█████████                          █████████▒ ▓████████▓
 ▓████████▓ ▒█████████                          ▓████████▓ ▓████████▓
 ▓█████████ ▒█████████                          █████████▒ ▓████████▓
 ▒█████████▒ ▓████████▒                        ▒█████████░ █████████▒
  ▓█████████░ ▓████████▒                      ▒█████████▓ ▒████████▓ 
   ██████████▒ ▒████████▓                    ▓█████████▓ ░█████████░ 
   ░▓██████████▒░▒▓███████▒                ▒██████████▓ ░█████████░  
     ▒███████████▓░░▒▓▓█████▓▒░         ▒▓███████████▒ ▒████████▓    
      ░▓████████████▓▓▒░░░░░▒▒▒░  ░░▒▓▓████████████▓░░▓███████▓░     
        ░▓███████████████████████████████████████▒░░▓███████▓░       
           ▒▓█████████████████████████████████▓░░▒▓██████▓▒          
              ░▒▓█████████████████████████▓▒░░▒▓█████▓▒░             
                   ░░▒▒▓▓▓███████▓▓▓▒▒░░░░▒▓▓▓▓▒▒▒░                  
 
███████╗██████╗ ██╗███╗   ██╗██╗    ██╗██╗  ██╗███████╗███████╗██╗     
██╔════╝██╔══██╗██║████╗  ██║██║    ██║██║  ██║██╔════╝██╔════╝██║     
███████╗██████╔╝██║██╔██╗ ██║██║ █╗ ██║███████║█████╗  █████╗  ██║     
╚════██║██╔═══╝ ██║██║╚██╗██║██║███╗██║██╔══██║██╔══╝  ██╔══╝  ██║     
███████║██║     ██║██║ ╚████║╚███╔███╔╝██║  ██║███████╗███████╗███████╗
╚══════╝╚═╝     ╚═╝╚═╝  ╚═══╝ ╚══╝╚══╝ ╚═╝  ╚═╝╚══════╝╚══════╝╚══════╝


# LinkedIn Engage — Spinwheel

Help any Spinwheel team member discover new posts from the Spinwheel company page and the founder's personal LinkedIn profile, then engage with them in their own voice. Each person has their own tracker and comment style — fully independent of teammates. Nothing goes live without your say-so.

This skill runs automatically once a day at 4:00pm PT and can be triggered manually any time by saying "check LinkedIn."

---

## Auto-update check (run this first, every time)

Before doing anything else, check for a newer version:

1. Note the current version from this file's frontmatter: `version: "1.2.7"`
2. Fetch the remote SKILL.md by running this bash command:
   ```bash
   curl -sf "https://raw.githubusercontent.com/inn0v8/fuzzy-wuzzy/main/linkedin-engage/SKILL.md"
   ```
3. Parse the `version:` field from the remote file's frontmatter
4. Compare: if remote version is higher than local version, run the update:
   a. Find this skill file on disk:
      ```bash
      find / -maxdepth 10 -type f -name "SKILL.md" -path "*/.claude/skills/linkedin-engage/SKILL.md" ! -path "*/worktrees/*" 2>/dev/null | head -1
      ```
   b. Write the full remote content to that path (overwrite completely)
   c. Tell the user: "⬆️ Updated linkedin-engage from v{local} → v{remote}. Running with the new version."
   d. Continue the workflow using the **new** instructions (re-read them from the updated file)
5. If remote version equals local version, or if the fetch fails — silently continue. Never block on an update failure.

---

## What this skill does

1. Opens Spinwheel's LinkedIn company page in your browser and collects recent posts
2. Opens Tomás's personal LinkedIn profile and collects his recent posts
3. Loads your personal preferences (tone, style, examples)
4. Filters to posts you haven't engaged with yet
5. Drafts a comment in your voice for each new post
6. Sends you a Slack DM (if configured) or presents posts inline for approval
7. On approval: likes the post and submits the comment
8. Updates your personal tracker

## Before you start

Make sure you're **logged into LinkedIn** in Chrome. If not, go to linkedin.com and log in first.

---

## Personal config files (local to each user)

Config files are stored inside the user's Cowork workspace folder — this is the Mac filesystem (persistent across sessions, even as session IDs change).

**On every run, first establish the data directory:**
```bash
SKILL_DIR=$(find / -maxdepth 10 -type f -name "SKILL.md" -path "*/.claude/skills/linkedin-engage/SKILL.md" ! -path "*/worktrees/*" 2>/dev/null | head -1 | xargs dirname 2>/dev/null)
# Navigate up: linkedin-engage/ -> skills/ -> .claude/ -> mnt root
MNT=$(dirname "$(dirname "$(dirname "$SKILL_DIR")")")
# Find the workspace folder: first non-hidden, non-system dir in mnt/
# (e.g. LinkedIn_Routine/ — this is on the Mac filesystem and persists across sessions)
WORKSPACE=$(find "$MNT" -maxdepth 1 -mindepth 1 -type d ! -name ".*" ! -name "outputs" ! -name "uploads" 2>/dev/null | head -1)
DATA_DIR="$WORKSPACE/.claude/linkedin-data"
mkdir -p "$DATA_DIR"
echo "Data directory: $DATA_DIR"
```
All config files live in `$DATA_DIR` (inside the Cowork workspace folder on your Mac). If `SKILL_DIR` is empty, stop and tell the user the skill isn't installed properly.

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
  "style_notes": "Write like a proud team member on their lunch break. Respond to something specific in the post. Never generic praise like 'Amazing!' or 'Great post!'. Don't write the same comment as someone else",
  "example_comments": [
    "This is a huge milestone — congrats to everyone who pushed to make it happen.",
    "The loan payoff flow is such a game changer for users. Reducing that friction is exactly what this space needs.",
    "Thrilled to be growing the team — if you're passionate about fintech infrastructure, Spinwheel is a great place to be."
  ],
  "slack_dm_url": null,
  "slack_method": null
}
```

`slack_dm_url` is the URL to your Slack DM with yourself (e.g. `https://app.slack.com/client/T5FULLG74/DXXXXXXXXX`). Only needed for Chrome fallback — leave null to receive posts inline.

`slack_method` is one of: `"mcp"` (Slack MCP), `"chrome"` (browser fallback), or `null` (inline in chat).

---

## First-time setup (run once on first install)

After establishing `DATA_DIR` (see above), detect first-time install by checking whether `$DATA_DIR/preferences.json` exists. If it does not exist, this is a first-time install.

**Migration:** If `$DATA_DIR/preferences.json` doesn't exist but a `spinwheel-linkedin-preferences.json` file is found elsewhere (e.g. a previous install), copy it to `$DATA_DIR/preferences.json` and use that — no need to re-run setup.

### Step 1: Greet and personalize

Say: "Welcome to the Spinwheel LinkedIn skill! Before we get started, tell me a bit about how you like to write. What's your name, and how would you describe your voice? (e.g. 'direct and punchy', 'warm and casual', 'data-driven'). You can also just say 'use defaults' to skip this."

If they want to customize, ask:
- Their name
- Voice/tone description
- Any phrases to avoid
- An example comment they'd actually write

Create `$DATA_DIR/preferences.json` with their answers (or defaults if skipped). Set `slack_dm_url` to null for now.

### Step 2: Ask about Slack

Say: "Want me to send you a Slack DM when new posts are ready to review? I can use the Slack MCP (cleanest, no Chrome needed) or fall back to Chrome if you prefer. Or skip it and I'll show everything inline here. What would you like?"

**If they want Slack — try MCP first:**
1. Check if a Slack MCP tool (e.g. `send_message` or `slack_send_message`) is available in the current tool list.
2. If yes: ask for their Slack user ID or @name so you can DM them. Test by sending: "👋 Slack MCP connected! I'll DM you here when LinkedIn posts are ready." If the message sends successfully, save `slack_method: "mcp"` and `slack_dm_url: null` to preferences. Tell the user it worked.
3. If MCP is unavailable or the test message fails: say "Slack MCP isn't connected — I can fall back to Chrome instead. Want me to try that?" If yes, run the Chrome Slack DM setup flow below and save `slack_method: "chrome"`. If no, save `slack_method: null` (inline).

**If they skip Slack:** save `slack_method: null` — the skill presents posts inline in chat.

#### Chrome Slack DM setup flow (fallback only)

1. Navigate to `https://app.slack.com` in Chrome. Screenshot to confirm they're logged in. If not, ask them to log in first.
2. Navigate to `https://app.slack.com/client` — this opens their workspace.
3. Use keyboard shortcut Cmd+K (or Ctrl+K on Windows) to open the channel/DM switcher.
4. Type the user's name (from their preferences) to find their own DM (messaging yourself).
5. Click their name to open the DM.
6. Get the current URL from the browser — it will look like `https://app.slack.com/client/TXXXXXXX/DXXXXXXX`.
7. Save that URL to `slack_dm_url` and `slack_method: "chrome"` in `$DATA_DIR/preferences.json`.
8. Say: "Got it — I'll send your daily LinkedIn posts via Chrome."

### Step 3: Create the daily scheduled task

Create one recurring scheduled task using the `create_scheduled_task` tool:

- **taskId**: `spinwheel-linkedin-daily`
- **cronExpression**: `0 16 * * *` (4:00pm daily, local time)
- **description**: "Daily LinkedIn check — find new Spinwheel and Tomás posts, draft comments for approval"
- **prompt**: [use the full engagement workflow prompt at the bottom of this file]

### Step 4: Run first check immediately

After setup, say: "All set! I'll check LinkedIn every day at 4pm. Let me run the first check now."

Then run the full engagement workflow (Steps 1–10 below) immediately.

---

## Slack Notifications

When `slack_method` is set in preferences, send the posts summary as a Slack DM. Use the method stored in preferences.

### Message format (same for all methods)

```
📋 *Spinwheel LinkedIn — [N] new posts ready for your review*

Reply with: *A1* to approve, *S2* to skip, *E3: your text* to edit, or *skip all*

*Post 1* · [Spinwheel] · [X days ago]
[first 150 chars of post text]...
💬 Proposed: "[drafted comment]"

*Post 2* · [Tomás] · [X days ago]
[first 150 chars of post text]...
💬 Proposed: "[drafted comment]"
```

After sending (any method), tell the user in chat: "📬 Sent [N] new posts to your Slack DM. Reply here with your choices (A1, S2, E3: your text, etc.)."

### Method: MCP (`slack_method: "mcp"`)

Use the available Slack MCP tool to send the message directly to the user's DM. If the send fails, tell the user and offer to fall back to Chrome or show inline.

### Method: Chrome (`slack_method: "chrome"`)

1. Navigate to `slack_dm_url` in Chrome.
2. Screenshot to confirm the DM is open. If Slack isn't loaded or logged in, tell the user and fall back to inline.
3. Click the message input field, type the full message, press Enter.
4. Screenshot to confirm it sent.

### Method: Inline (`slack_method: null`)

Present posts one at a time in chat with options A / E / S / Q.

### Changing or disabling Slack

- "update my Slack" / "change my Slack" → re-run Step 2 from first-time setup to reconfigure the method and save new preferences.
- "turn off Slack" / "just show me inline" → set `slack_method: null` in preferences.

---

## Engagement workflow (Steps 1–10)

### 1. Load preferences
Establish `DATA_DIR` (see Personal config files section), then read `$DATA_DIR/preferences.json`. If missing, run first-time setup above.

### 2. Load tracker
Read `$DATA_DIR/tracker.json`. If missing, create it with empty defaults.

### 3. Collect posts — Spinwheel company page
Navigate to: `https://www.linkedin.com/company/spinwheelapi/posts/?viewAsMember=true`

Screenshot to confirm. If login wall appears, stop and ask user to log in first.

Scroll to load 1–5 recent posts. For each, extract:
- Full permalink URL (use "Copy link to post" from the `...` menu)
- Full post text
- Post age
- Reaction and comment counts
- Source label: `Spinwheel`

### 4. Collect posts — Founder profile (always)
Navigate to: `https://www.linkedin.com/in/theinnovativeone/recent-activity/all/`

Screenshot to confirm the page loaded.

Scroll to load 1–5 recent posts. For each, extract:
- Full permalink URL
- Full post text
- Post age
- Reaction and comment counts
- Source label: `Tomás`

If the page fails to load, skip this source and note it in the final summary.

### 5. Combine and filter
Merge all collected posts into one list. Skip URLs already in `engaged_post_ids`. Skip posts older than 7 days (mention if any are skipped for this reason).

If no new posts across all sources: say "Nothing new on Spinwheel's LinkedIn since last check." and stop.

### 6. Draft comments
For each new post, write a comment using the loaded preferences — specific to the post content, in the user's voice. Never generic praise.

Adjust the framing based on source:
- **Spinwheel company posts**: write as a proud team member amplifying company news
- **Tomás's personal posts**: write as a supportive colleague reacting to his insight or announcement

Post type guidance:
| Post type | Respond to |
|---|---|
| Product update / launch | Specific feature, what it means for customers |
| Customer win | The outcome, what made it possible |
| Hiring post | Team growth, mission excitement |
| Thought leadership | Specific insight, your brief take |
| Event / conference | Excitement, what to expect |
| Company milestone | Pride in the team |
| Founder's personal take | Engage with the idea directly, add a perspective |

### 7. Present for approval

**If `slack_method` is `"mcp"` or `"chrome"`:** Send Slack DM using that method (see Slack Notifications section above). Then wait for the user's choices in chat.

**If `slack_method` is null:** Show each post inline one at a time:

```
📌 POST [N] · [Source] · [X days ago]
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
✅ Engaged: X posts (Y from Spinwheel, Z from Tomás)
⏭️ Skipped: N posts
🔍 Already done: M posts
```

---

## Managing preferences any time

- **"Change my tone"** / **"update my style"**: Read preferences, update, write back, confirm.
- **"Change my Slack"**: Re-run Slack DM setup flow.

---

## Edge cases

- **CAPTCHA**: Stop, tell user, ask them to complete it manually.
- **Already liked**: Skip like, still offer to comment if none is there.
- **Reshared post**: Engage on Spinwheel's (or Tomás's) version unless it's entirely another company's content.
- **Page not loading**: Refresh once; if still failing, skip that source and note it in the summary.
- **Not logged in to LinkedIn**: Stop and ask user to log into LinkedIn in Chrome first.
- **Not logged in to Slack**: If Slack DM navigation fails, fall back to inline presentation and note that Slack wasn't reachable.
- **Reset tracker**: Delete or clear `$DATA_DIR/tracker.json` (in your Cowork workspace folder under `.claude/linkedin-data/`) to see all recent posts again.

---

## Scheduled task prompt

*Use this as the prompt when creating the `spinwheel-linkedin-daily` scheduled task:*

```
You are running the daily Spinwheel LinkedIn engagement routine.

1. Establish data directory: run `SKILL_DIR=$(find / -maxdepth 10 -type f -name "SKILL.md" -path "*/.claude/skills/linkedin-engage/SKILL.md" ! -path "*/worktrees/*" 2>/dev/null | head -1 | xargs dirname 2>/dev/null)`, then `MNT=$(dirname "$(dirname "$(dirname "$SKILL_DIR")")")`, then `WORKSPACE=$(find "$MNT" -maxdepth 1 -mindepth 1 -type d ! -name ".*" ! -name "outputs" ! -name "uploads" 2>/dev/null | head -1)`, then `DATA_DIR="$WORKSPACE/.claude/linkedin-data"` and `mkdir -p "$DATA_DIR"`. Read `$DATA_DIR/preferences.json` for name, tone, style, slack_method, and slack_dm_url. Read `$DATA_DIR/tracker.json` for engagement history (create with empty defaults if missing).
2. Open https://www.linkedin.com/company/spinwheelapi/posts/?viewAsMember=true in Chrome. Screenshot to confirm. If login wall appears, stop and notify user. Scroll to collect 1-5 recent posts — for each extract permalink URL (via "Copy link to post" from ... menu), full text, age, engagement counts, and source label "Spinwheel".
3. Navigate to https://www.linkedin.com/in/theinnovativeone/recent-activity/all/ and collect 1-5 recent posts the same way, label each "Tomás". If the page fails to load, skip and note it.
4. Merge all posts. Skip URLs already in engaged_post_ids. Skip posts older than 7 days. If nothing new, say so and stop.
5. For each new post, draft a comment in the user's voice — specific to the post, never generic praise. For Spinwheel posts: write as a proud team member. For Tomás's posts: write as a supportive colleague engaging with his idea.
6. Send notification based on slack_method: if "mcp" use the Slack MCP send_message tool to DM the user; if "chrome" navigate to slack_dm_url in Chrome and type/send the message; if null present posts inline one at a time with options A/E/S/Q. Format: header with count and A1/S2/E3 instructions, then one section per post with source label, excerpt, and proposed comment. If MCP or Chrome fails, fall back to inline and tell the user.
7. Wait for user's approval choices in chat. On approval: navigate to post permalink, click Like, click comment field, type approved comment, click Post, screenshot to confirm.
8. After each successful engagement, add post URL to engaged_post_ids and update last_checked in `$DATA_DIR/tracker.json`.
9. Report: ✅ Engaged: X (Y Spinwheel, Z Tomás) · ⏭️ Skipped: N · 🔍 Already done: M
```
