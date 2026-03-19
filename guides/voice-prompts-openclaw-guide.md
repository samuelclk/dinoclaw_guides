# Practical Guide: Enabling Voice Prompts in OpenClaw

This is the simple version of what we learned while trying to make voice notes work well with OpenClaw.

The goal is:

- speak into Telegram using a voice note
- have OpenClaw turn that into text
- let the agent respond to the transcribed prompt
- keep the setup understandable for normal humans

---

## What we learned

There are **two main ways** to transcribe voice notes in OpenClaw:

### Option 1: Local transcription
This means your own machine does the speech-to-text work.

Examples:
- Python Whisper
- whisper.cpp / `whisper-cli`

### Option 2: Cloud transcription
This means a third-party provider does the speech-to-text work.

Example:
- Deepgram

---

## The short conclusion

For this setup, the best overall result was:

- **Deepgram (`big`)** for everyday voice prompting
- **Python Whisper medium (`med`)** when privacy matters more
- **Python Whisper small (`small`)** if you want faster local transcription and can accept lower accuracy

In practice:
- **Deepgram was fastest**
- **local Whisper was more private**
- **whisper.cpp was fast but not accurate enough for exact wording in our tests**

---

# Part 1 — How voice prompts work in OpenClaw

OpenClaw does not magically understand voice notes by default just because Telegram supports voice messages.

You need to enable **audio transcription** in `openclaw.json`.

The relevant config section is:

```json
"tools": {
  "media": {
    "audio": {
      "enabled": true,
      "models": [
        {
          "type": "cli",
          "command": "whisper",
          "args": [
            "--model", "small",
            "--language", "English",
            "--output_format", "txt",
            "--output_dir", "{{OutputDir}}",
            "{{MediaPath}}"
          ],
          "timeoutSeconds": 120
        }
      ]
    }
  }
}
```

When this is enabled, OpenClaw:
1. receives the voice note
2. transcribes it
3. injects the transcript into the message pipeline
4. lets the assistant respond to the transcript as a normal prompt

---

# Part 2 — The three presets we ended up with

We created three simple aliases:

## `small`
Uses:
- Python Whisper `small`

Good for:
- local transcription
- faster turnaround than medium

Tradeoff:
- less accurate than medium

---

## `med`
Uses:
- Python Whisper `medium`

Good for:
- local transcription
- better fidelity than small

Tradeoff:
- slower than small

---

## `big`
Uses:
- Deepgram `nova-3`

Good for:
- best speed
- best usability for everyday voice prompts

Tradeoff:
- your audio is sent to a third-party provider

---

# Part 3 — The benchmark results we observed

From our tests:

- **Python Whisper `small`** → about **31 seconds**
- **Python Whisper `medium`** → about **43 seconds**
- **Deepgram `nova-3`** → about **10 seconds**

So Deepgram was the clear speed winner.

---

# Part 4 — Privacy tradeoffs

## Local Whisper
Privacy level:
- **better**

Why:
- audio stays on your own machine
- no third-party speech-to-text provider processes it

Best for:
- sensitive notes
- secrets
- private/confidential prompts

---

## Deepgram
Privacy level:
- **less private than local**

Why:
- your voice note is sent to Deepgram for transcription

Best for:
- normal app-building workflows
- routine prompts
- speed and convenience

---

## Recommended rule

Use:
- **`big`** for everyday workflow voice notes
- **`med`** for private/sensitive voice notes

That gives you a practical balance between convenience and privacy.

---

# Part 5 — How to add your Deepgram API key

If you want to use the `big` preset, put your Deepgram API key in:

```bash
/home/huatyou/.openclaw/.env
```

Add a line like this:

```bash
DEEPGRAM_API_KEY=your_actual_key_here
```

### Step-by-step

```bash
cd /home/huatyou/.openclaw
nano .env
```

Add the line.

Then save and exit:
- `CTRL+O`
- `ENTER`
- `CTRL+X`

---

# Part 6 — The preset files

These preset files live here:

```bash
/home/huatyou/.openclaw/guides/voice-stt-presets/
```

Files:
- `small.json`
- `med.json`
- `big.json`

They correspond to:
- `small` = Python Whisper small
- `med` = Python Whisper medium
- `big` = Deepgram

## Actual preset JSON

### `small.json`

```json
{
  "enabled": true,
  "models": [
    {
      "type": "cli",
      "command": "whisper",
      "args": [
        "--model", "small",
        "--language", "English",
        "--output_format", "txt",
        "--output_dir", "{{OutputDir}}",
        "{{MediaPath}}"
      ],
      "timeoutSeconds": 120
    }
  ]
}
```

### `med.json`

```json
{
  "enabled": true,
  "models": [
    {
      "type": "cli",
      "command": "whisper",
      "args": [
        "--model", "medium",
        "--language", "English",
        "--output_format", "txt",
        "--output_dir", "{{OutputDir}}",
        "{{MediaPath}}"
      ],
      "timeoutSeconds": 180
    }
  ]
}
```

### `big.json`

```json
{
  "enabled": true,
  "models": [
    {
      "provider": "deepgram",
      "model": "nova-3"
    }
  ]
}
```

---

# Part 7 — How switching presets works

We created a helper script so you do not have to manually edit `openclaw.json` every time.

Script:

```bash
/home/huatyou/.openclaw/scripts/use-stt-preset.sh
```

Usage:

```bash
/home/huatyou/.openclaw/scripts/use-stt-preset.sh small
/home/huatyou/.openclaw/scripts/use-stt-preset.sh med
/home/huatyou/.openclaw/scripts/use-stt-preset.sh big
```

What it does:
1. applies the preset to `openclaw.json`
2. restarts the gateway
3. polls until the gateway is confirmed working or failed
4. if confirmation is delayed, it reports that it is still waiting
5. finally reports ready / not ready

This is important because restarting the gateway is not enough by itself — you want confirmation that the system actually came back successfully.

---

# Part 8 — Why we made a wrapper script

We learned the hard way that after changing the voice-transcription config, the gateway must be restarted and then **confirmed**.

So the correct behavior is:
- do not just restart and assume success
- poll until you know whether it worked
- if it takes longer than expected, keep updating the user
- then send a final ready / not ready result

That behavior is now documented in:
- `workspace/AGENTS.md`
- `guides/voice-stt-presets/README.md`
- `scripts/use-stt-preset.sh`

---

# Part 9 — Why we do NOT store Whisper model files in GitHub

We also learned not to push local Whisper model binaries directly to GitHub.

Bad idea:
- they are huge
- Git handles them badly
- pushes become slow or hang
- GitHub is not the right place for these binary assets

Instead:
- keep model `.bin` files local on disk
- ignore them in `.gitignore`
- store a guide explaining how to download them

That guide is here:

```bash
/home/huatyou/.openclaw/guides/whisper-model-downloads.md
```

---

# Part 10 — What to use in real life

## Recommended default setup

For a normal person who wants voice prompts that feel usable:

### Everyday use
Use:
```bash
/home/huatyou/.openclaw/scripts/use-stt-preset.sh big
```

Why:
- fastest
- easiest
- best for everyday app-building prompts

### Sensitive/private use
Use:
```bash
/home/huatyou/.openclaw/scripts/use-stt-preset.sh med
```

Why:
- local transcription
- better privacy
- better wording fidelity than local small

### If you want faster local but can tolerate more mistakes
Use:
```bash
/home/huatyou/.openclaw/scripts/use-stt-preset.sh small
```

---

# Part 11 — Simple recommendation for laymen

If you don’t want to overthink it:

- **Use `big` most of the time**
- **Use `med` when the voice note is sensitive**
- ignore `small` unless you specifically care about faster local-only transcription

That is the simplest useful policy.

---

# Part 12 — Final takeaway

What we discovered was simple:

- local transcription is possible
- but speed and accuracy fight each other on a CPU-only box
- cloud transcription is much faster
- Deepgram currently looks like the best practical everyday option for voice prompting
- local Whisper still matters for privacy-sensitive prompts

So the final working model is:

- **`big` = convenience**
- **`med` = privacy**
- **`small` = local speed compromise**

That’s the practical answer.
