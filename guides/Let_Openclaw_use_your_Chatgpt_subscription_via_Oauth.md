# Let Openclaw use your Chatgpt subscription via Oauth

1. Create an SSH tunnel from your working laptop into your Openclaw machine.

```bash
ssh -L 1455:127.0.0.1:1455 USER@IP_ADDRESS -p PORT -i IDENTITY
```

2. Run `openclaw config`.
3. Then select:
   - `Local (this machine)`
   - `Model`
   - `OpenAI (Codex OAuth + API key)`
   - `OpenAI Codex (ChatGPT OAuth)`
4. An OAuth URL will be generated. Paste it into the browser on your working laptop.
5. Sign in to your Chatgpt account and you will be redirected to a “Success” page.
6. Copy the URL of the “Success” page (redirect URL) and paste it into the Openclaw configuration terminal.
7. Select `openai-codex/gpt-5.4` as your model.
