# Meeting Summary Style Training

Use this folder to build paired examples for Dino's meeting-summary rewrite workflow.

## Local folders

- Source: `/home/lidoclaw/.openclaw/workspace/meeting-summary-style-training/source`
- Target: `/home/lidoclaw/.openclaw/workspace/meeting-summary-style-training/target`
- Negative: `/home/lidoclaw/.openclaw/workspace/meeting-summary-style-training/negative`

## Google Drive mapping

Parent folder:
- `Lido/Meeting Summary Style Training`
- ID: `1p706bSHmq1Bzdv5j--BJhafY5IgO0l8r`
- URL: <https://drive.google.com/drive/folders/1p706bSHmq1Bzdv5j--BJhafY5IgO0l8r>

Child folders:
- Source
  - Drive path: `Lido/Meeting Summary Style Training/Source`
  - ID: `1l_Y_eNnx5NUbcOwl0CfFOgamsFfUVNd9`
  - URL: <https://drive.google.com/drive/folders/1l_Y_eNnx5NUbcOwl0CfFOgamsFfUVNd9>
- Target
  - Drive path: `Lido/Meeting Summary Style Training/Target`
  - ID: `1FjaLsL4cGSk_Hpyhsx9KCEiYeoIBxmKz`
  - URL: <https://drive.google.com/drive/folders/1FjaLsL4cGSk_Hpyhsx9KCEiYeoIBxmKz>
- Negative
  - Drive path: `Lido/Meeting Summary Style Training/Negative`
  - ID: `1M9IMg6uayaRp2S1SdajLGQGACLqEQftl`
  - URL: <https://drive.google.com/drive/folders/1M9IMg6uayaRp2S1SdajLGQGACLqEQftl>

## Intended use

- Put raw transcripts or Gemini notes into `source/` / Drive `Source`
- Put your final handwritten summaries into `target/` / Drive `Target`
- Put bad outputs / anti-pattern examples / too-Gemini-ish summaries into `negative/` / Drive `Negative`
- Prefer matching filenames so pairing is easy, e.g.:
  - `2026-03-18-product-sync.md` in Source
  - `2026-03-18-product-sync.md` in Target
  - `2026-03-18-product-sync.md` in Negative when you want a contrast example

Optional suffix pattern if you want separation within a filename pair:
- `2026-03-18-product-sync.source.md`
- `2026-03-18-product-sync.target.md`
