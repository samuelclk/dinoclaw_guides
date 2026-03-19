# Local Whisper Model Downloads

Do **not** commit Whisper model binaries to Git. Keep them local on disk and re-download them when needed.

## Model storage location

Save Whisper model files here:

```bash
/home/huatyou/.openclaw/models/whisper/
```

Create the folder if needed:

```bash
mkdir -p /home/huatyou/.openclaw/models/whisper
```

---

## Model files currently used in this setup

### 1. `ggml-base.en.bin`

```bash
cd /home/huatyou/.openclaw/models/whisper
curl -L --fail -o ggml-base.en.bin https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-base.en.bin
```

### 2. `ggml-small.en.bin`

```bash
cd /home/huatyou/.openclaw/models/whisper
curl -L --fail -o ggml-small.en.bin https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-small.en.bin
```

### 3. `ggml-medium.en.bin`

```bash
cd /home/huatyou/.openclaw/models/whisper
curl -L --fail -o ggml-medium.en.bin https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-medium.en.bin
```

---

## What each model is for

- **base.en** — fastest, weakest fidelity
- **small.en** — better balance of speed vs quality
- **medium.en** — higher fidelity, slower

---

## Optional: set the model path in `.env`

Open:

```bash
cd /home/huatyou/.openclaw
nano .env
```

Add or update:

```bash
WHISPER_CPP_MODEL=/home/huatyou/.openclaw/models/whisper/ggml-small.en.bin
```

Save and exit `nano`:

- `CTRL+O`
- `ENTER`
- `CTRL+X`

---

## After changing model paths or config

Restart the gateway:

```bash
systemctl --user restart openclaw-gateway
systemctl --user status openclaw-gateway --no-pager
```

---

## Important rule

These `.bin` files should remain:

- **local on disk**
- **ignored by Git**
- **documented in this guide**

They should **not** be committed to the backup repo.
