# Discord Setup for Persistent Thread-Bound Sub-Agents on OpenClaw

> Doc index: `workspace/guides/Meeting_Notes_Summariser_Index.md`
> GitHub: <https://github.com/stakesaurus-lido/lidoclaw/blob/main/workspace/guides/Meeting_Notes_Summariser_Index.md>

This guide shows how to set up Discord so OpenClaw can use **thread-bound sub-agents / ACP workers** reliably.

It is written for an operator who wants:
- a private Discord workspace
- one or more Discord channels dedicated to specialist workflows
- persistent thread-bound worker sessions
- strict access control so only the owner can use the bot

---

## What this setup gives you

With Discord configured correctly, OpenClaw can do things that were blocked on Telegram in this environment:

- bind a Discord thread to a spawned sub-agent session
- bind a Discord thread to a persistent ACP worker
- keep follow-up messages in that thread routed to the same worker
- let you create topic-specific specialist lanes like:
  - `#meeting-notes`
  - `#coding-worker`
  - `#research`

This is the cleanest way to get persistent specialist workflows on OpenClaw chat surfaces.

---

## Prerequisites

You need:
- a machine already running OpenClaw
- access to edit `/home/lidoclaw/.openclaw/.env`
- access to edit `/home/lidoclaw/.openclaw/openclaw.json`
- a Discord account
- permission to create a Discord server or add a bot to one

---

## Recommended architecture

For best results, use:
- **one private Discord server** for Dino / OpenClaw work
- **one dedicated channel** for each specialist workflow
- optional **forum channels** if you want one thread per task/topic
- OpenClaw allowlists so only your Discord account can interact with the bot

Recommended first channel:
- `#meeting-notes`

---

# Part 1 - Create and install the Discord bot

## 1. Create the application

1. Open the **Discord Developer Portal**
2. Click **New Application**
3. Give it a name
4. Open the **Bot** page
5. Click **Add Bot**

## 2. Enable the required intents

Still on the **Bot** page, enable:
- **Message Content Intent** *(required)*
- **Server Members Intent** *(recommended)*

You do **not** need OAuth2 Code Grant for this workflow.
Leave **Requires OAuth2 Code Grant** off.

## 3. Install the bot into your server

Use **OAuth2 -> URL Generator**.

Select scopes:
- `bot`
- `applications.commands`

Select permissions:
- View Channels
- Send Messages
- Read Message History
- Embed Links
- Attach Files
- Add Reactions *(optional)*

Then open the generated URL and add the bot to your server.

### If Discord complains about private/public app settings
A non-public bot can still work fine if you install it correctly into your own server.
You do not need to make it public just to use OpenClaw privately.

---

# Part 2 - Collect the required IDs

## 4. Enable Developer Mode in Discord

In Discord:
1. Open **User Settings**
2. Go to **Advanced**
3. Turn on **Developer Mode**

## 5. Copy the IDs you need

### Server ID
- Right-click the server icon
- Click **Copy Server ID**

### Channel ID
- Right-click the channel you want OpenClaw to use
- Click **Copy Channel ID**

If using a forum channel:
- right-click the forum itself for the parent forum ID
- right-click a specific thread if you want the thread ID

### Your User ID
- Right-click your username/avatar
- Click **Copy User ID**

These are long numeric strings like:
- `123456789012345678`

---

# Part 3 - Store the bot token in .env

## 6. Add the token to `.env`

On the host running OpenClaw:

```bash
cd /home/lidoclaw/.openclaw
nano .env
```

Add:

```bash
DISCORD_BOT_TOKEN=your_real_discord_bot_token_here
```

Save and exit:
- `CTRL+O`
- `ENTER`
- `CTRL+X`

### Important
- Do **not** paste the bot token into chat
- Treat `.env` as sensitive
- Do not commit `.env` to git

---

# Part 4 - Add the Discord channel config to openclaw.json

## 7. Minimal secure Discord config

Add a `channels.discord` block using:
- env-backed token
- guild allowlist
- channel allowlist
- user allowlist

Example shape:

```json5
{
  "channels": {
    "discord": {
      "enabled": true,
      "token": "${DISCORD_BOT_TOKEN}",
      "groupPolicy": "allowlist",
      "guilds": {
        "YOUR_SERVER_ID": {
          "users": ["YOUR_USER_ID"],
          "channels": {
            "YOUR_CHANNEL_ID": {
              "allow": true,
              "requireMention": false
            }
          }
        }
      }
    }
  }
}
```

### What this does
- bot is enabled
- only the listed Discord server is trusted
- only the listed channel is allowed
- only your Discord user ID is allowed to talk to the bot there
- the bot responds without `@mention` in that channel

If you want stricter behavior, set:

```json5
"requireMention": true
```

---

# Part 5 - Enable thread bindings for persistent workers

This is the important part for persistent thread-bound sub-agents / ACP workers.

## 8. Add thread binding config

Add these sections:

```json5
{
  "session": {
    "threadBindings": {
      "enabled": true,
      "idleHours": 24,
      "maxAgeHours": 0
    }
  },
  "channels": {
    "discord": {
      "threadBindings": {
        "enabled": true,
        "idleHours": 24,
        "maxAgeHours": 0,
        "spawnSubagentSessions": true,
        "spawnAcpSessions": true
      }
    }
  }
}
```

## What these fields mean

### `session.threadBindings.enabled`
Turns on global thread-binding support.

### `channels.discord.threadBindings.enabled`
Allows Discord threads to bind to sessions.

### `spawnSubagentSessions`
Required if you want:
- `sessions_spawn(... thread: true ...)`
- automatic thread creation/binding for sub-agents

### `spawnAcpSessions`
Required if you want:
- persistent ACP workers bound to Discord threads
- Codex / Claude Code / Gemini ACP sessions launched with `thread: true`

### `idleHours`
How long an inactive thread binding stays attached before auto-unfocus.

### `maxAgeHours`
Hard cap on binding lifetime.
- `0` means no hard cap.

---

# Part 6 - Choose the right Discord binding pattern

There are **two different patterns**, and they are not interchangeable.

## Pattern A - Manager-routed channel (recommended for `msw` / Meeting Summary Worker)

Use a normal `type: "route"` binding when you want the main agent to:
- load the meeting-summary runbook
- load the canonical `.md` files
- decide whether to reuse an existing output
- explicitly spawn a specialist sub-agent with `sessions_spawn`
- stay in manager mode and report back with the final result

This is the **correct pattern** for `msw <file>` and `Use meeting summary worker: <file>`.

## Pattern B - Persistent ACP channel binding

Use a top-level `type: "acp"` binding only if you want one Discord channel to always map straight into the same persistent ACP worker.

This is useful for a true always-on coding/specialist lane, but it is **not** the right default for the Meeting Summary Worker flow if you want manager → spawned sub-agent behavior.

## 9. Add an ACP agent definition

Example shape:

```json5
{
  "agents": {
    "list": [
      {
        "id": "codex",
        "runtime": {
          "type": "acp",
          "acp": {
            "agent": "codex",
            "backend": "acpx",
            "mode": "persistent",
            "cwd": "/home/lidoclaw/.openclaw/workspace"
          }
        }
      }
    ]
  }
}
```
```

## 10. Bind the Discord channel the right way for your workflow

### Option A - Recommended for Meeting Summary Worker (`msw`)

Use a normal route binding:

```json5
{
  "bindings": [
    {
      "type": "route",
      "agentId": "codex",
      "match": {
        "channel": "discord",
        "accountId": "default",
        "peer": {
          "kind": "channel",
          "id": "YOUR_CHANNEL_ID"
        }
      }
    }
  ]
}
```

### What this gives you
Messages in that Discord channel route to the main manager agent first.
That lets `msw <file>` do the full proven flow:
- load `meeting-summary-style-training/RUNBOOK.md`
- load the canonical style/rubric/prompt/spec docs
- find the source file
- call `sessions_spawn(runtime="subagent", ...)`
- wait for the child result
- return only the final Samuel-style summary

### Verified lesson
Do **not** use a direct `type: "acp"` binding for the Meeting Summary Worker lane if your goal is manager → spawned sub-agent behavior.
A direct ACP binding can bypass the manager path and prevent `msw` from doing the runbook-driven worker orchestration you actually want.

### Option B - Persistent ACP lane

If you truly want one Discord channel to always map into the same persistent ACP workspace, use:

```json5
{
  "bindings": [
    {
      "type": "acp",
      "agentId": "codex",
      "match": {
        "channel": "discord",
        "accountId": "default",
        "peer": {
          "kind": "channel",
          "id": "YOUR_CHANNEL_ID"
        }
      },
      "acp": {
        "label": "codex-main"
      }
    }
  ]
}
```

Use that pattern for an always-on ACP workspace, not for the default `msw` meeting-summary lane.

---

# Part 7 - Restart the gateway and verify

## 11. Restart the gateway

Use the OpenClaw CLI on the host.
If the binary is not on PATH, use its absolute path.

Example:

```bash
/home/huatyou/.npm-global/bin/openclaw gateway restart
```

## 12. Verify the bot is live

Check that:
- the gateway starts cleanly
- the Discord bot appears online in your server
- the bot responds in the allowlisted channel

## 13. Test a normal message

In the Discord channel, send:

```text
hello
```

If `requireMention` is `false`, the bot should respond.
If `requireMention` is `true`, send:

```text
@BotName hello
```

---

# Part 8 - Test thread-bound worker behavior

## 14. Test a thread-bound specialist session

Once thread bindings are enabled, you can use Discord-native workflows like:
- `/focus <target>`
- `/unfocus`
- `/agents`

Or you can have OpenClaw spawn a worker with `thread: true`.

## Expected behavior
- a thread is created or bound
- follow-up messages inside that thread keep going to the same specialist session
- resets/new thread flows stay scoped to that worker

---

# Part 9 - Recommended setup for meeting summaries

For the meeting-summary workflow, recommended channel setup is:

## Option A - Simple dedicated channel
- `#meeting-notes`
- channel routed to the main manager agent with a `type: "route"` binding
- `msw <file>` triggers runbook load + canonical docs + spawned specialist sub-agent
- all summary requests happen there

## Option B - Forum channel
- forum parent = `Meeting Notes`
- one thread/post per transcript / summary task
- each thread can bind to a worker session

Option B is usually better if you want many summaries in parallel without mixing contexts.

## Quick decision table: manager-routed lane vs persistent ACP lane

| Question | Manager-routed lane (`type: "route"`) | Persistent ACP lane (`type: "acp"`) |
| --- | --- | --- |
| Who gets the message first? | Main manager agent | Persistent ACP worker |
| Best for | Runbook-driven orchestration | Long-lived direct specialist workspace |
| Can it load docs then decide what to do? | Yes | Not as the default manager step |
| Can it spawn a specialist sub-agent on demand? | Yes — this is the right pattern | Maybe, but that is not what the lane is optimized for |
| Best for `msw <file>`? | **Yes** | **No** |
| Best for always-on coding lane? | Sometimes | **Yes** |
| Best when you want QA / retries / manager oversight? | **Yes** | Usually no |
| Best when every message should hit the same worker directly? | No | **Yes** |

### Use manager-routed lane when:
- the task needs a runbook first
- the task needs doc loading first
- the agent should choose a workflow
- the agent should spawn a worker
- the main agent should stay in manager / QA mode

### Use persistent ACP lane when:
- you want one direct always-on workspace
- you want continuity inside the same specialist worker
- every message in the channel should go straight to that worker
- you do not need manager-first orchestration on every turn

---

# Operator commands to use later

Once Discord is working, use explicit commands like:

```text
msw blindtest_source_01_kiln
```

```text
Use meeting summary worker: blindtest_source_01_kiln
```

```text
msw blindtest_source_01_kiln and self-score it
```

```text
compare blindtest_source_01_kiln against blindtest_target_01_kiln
```

Verified good path for `msw` on Discord:
- main routed agent receives the request
- runbook + canonical docs load first
- source file is located
- `sessions_spawn(runtime="subagent")` is called
- child result comes back to the manager
- manager returns the final summary

If you want that behavior, keep the lane on a normal route binding rather than a direct ACP channel binding.

---

# Troubleshooting

## Bot installed but not responding
Check:
- bot token is correct in `.env`
- `channels.discord.enabled` is true
- guild ID is correct
- channel ID is correct
- your user ID is correct
- Message Content Intent is enabled
- gateway was restarted after config changes

## Bot responds nowhere in server
Check:
- `groupPolicy` is `allowlist`
- the guild is actually present under `channels.discord.guilds`
- the channel is listed under that guild
- you are listed under `users`

## Bot only works in DMs
Check guild allowlist and channel allowlist.
DM support is separate from guild-channel support.

## Persistent thread worker still not working
Check:
- `session.threadBindings.enabled=true`
- `channels.discord.threadBindings.enabled=true`
- `spawnSubagentSessions=true` for sub-agents
- `spawnAcpSessions=true` for ACP workers
- the lane is using the correct binding type for the workflow

For Meeting Summary Worker specifically:
- prefer `bindings[].type="route"`
- verify the manager loads `meeting-summary-style-training/RUNBOOK.md`
- verify the manager then calls `sessions_spawn(runtime="subagent", ...)`
- do not assume a direct `type="acp"` binding will preserve manager-mode orchestration

---

# Recommended secure baseline

Use this as your default posture:
- private Discord server
- allowlisted guild only
- allowlisted channel only
- allowlisted user only
- env-backed token only
- thread bindings enabled only on Discord
- one specialist workflow per channel/forum lane

---

# Summary

If you want persistent thread-bound workers on OpenClaw, **Discord is the right surface**.

The key pieces are:
1. Discord bot installed correctly
2. env-backed `DISCORD_BOT_TOKEN`
3. guild/channel/user allowlists
4. `session.threadBindings.enabled=true`
5. `channels.discord.threadBindings.enabled=true`
6. `spawnSubagentSessions=true` and/or `spawnAcpSessions=true`
7. choose the correct binding type for the lane
   - `type: "route"` for manager-led workflows like `msw`
   - `type: "acp"` only for true always-on ACP lanes

Verified learning from the work instance:
- Discord `msw <file>` worked correctly once the lane was switched from direct ACP binding to normal route binding
- the successful path was: Discord message → manager agent → runbook/docs load → `sessions_spawn` sub-agent → child completion → final summary reply

That combination gives you the cleanest specialist workflow architecture currently available in OpenClaw chat surfaces.
