# Token Usage Optimisation (OpenAI OAuth)

For personal reference only. Abstracted from @mattganzak’s (ScaleUp Media) OpenClaw Token Optimization Guide. All credits to Matt Ganzak and ScaleUp Media.

### Smart Session Initialisation Rules

1. Use a faster and cheaper model for general conversations
2. Create aliases so you can say "use basic" or "use pro" to switch models on-demand.

> Prompt your Openclaw bot with the following

Update my openclaw.json config file with the following.

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "openai/gpt-5.3-codex"
      },
      "models": {
        "openai/gpt-5.4-pro": {
          "alias": "pro"
        },
        "openai/gpt-5.3-codex": {
          "alias": "basic"
        }
      }
    }
  }
}
```

### Define Model Routing Rules

Add Routing Rules to [AGENTS.md](http://AGENTS.md) (system prompt)

> Prompt your Openclaw bot

Add these rules to your AGENTS.md.

```text
MODEL SELECTION RULE:
 
Default: Always use Codex 5.3 via Oauth
Switch to Codex 5.4 ONLY when:
- Architecture decisions
- Production code review
- Security analysis
- Complex debugging/reasoning
- Strategic multi-project decisions
 
When in doubt: Try Codex 5.3 via Oauth first.
```

1. Use a smaller OpenAI model for heartbeat checks
   1. Edit your openclaw.json config file

> Prompt your Openclaw bot

Update your openclaw.json to route heartbeats to Ollama by adding the following snippet under agents.defaults

```json
"heartbeat": {
  "every": "1h",
  "model": "openai/gpt-4.1-mini",
  "session": "heartbeat-main",
  "target": "telegram",
  "prompt": "Check: Any blockers, opportunities, or progress updates needed?",
  "lightContext": true
}
```

Keep all other config unchanged.

### Rate Limits & Budget Controls

Edit your AGENTS.md config file.

> Prompt your Openclaw bot

Add these rate limit rules to your AGENTS.md file.

```text
RATE LIMITS:
 
- 5 seconds minimum between API calls
- 10 seconds between web searches
- Max 5 searches per batch, then 2-minute break
- Batch similar work (one request for 10 leads, not 10 requests)
- If you hit 429 error: STOP, wait 5 minutes, retry
 
DAILY BUDGET: $5 (warning at 75%)
MONTHLY BUDGET: $200 (warning at 75%)
```

### Caching

Edit your openclaw.json config file

> Prompt your Openclaw bot

Update openclaw.json to enable Anthropic prompt caching by ensuring:

- `agents.defaults.models["openai/gpt-5.4-pro"].params.cacheRetention = "long"`
- `agents.defaults.models["openai/gpt-5.3-codex"].params.cacheRetention = "short"`
- `agents.defaults.contextPruning = { "mode": "cache-ttl", "ttl": "1h" }`

Keep all other existing config unchanged.
