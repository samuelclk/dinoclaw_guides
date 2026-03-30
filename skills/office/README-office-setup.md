# Office skill local runtime notes

Installed official Anthropic skill folders:
- `skills/pdf`
- `skills/docx`
- `skills/pptx`
- `skills/xlsx`

Local runtime support added:
- Python venv: `.venv-office`
- Node modules: `.office-node/node_modules`

Verified available:
- Python packages: `pypdf`, `pdfplumber`, `pillow`, `openpyxl`, `docx2python`
- Node packages: `docx`, `pptxgenjs`
- System tools: `libreoffice`, `soffice`, `pdftoppm`

Still missing / not yet installed:
- `pandoc`
- `tesseract`

Usage note:
- For Python-based helper scripts, activate `.venv-office` first.
- For Node-based DOCX/PPTX generation helpers, use `NODE_PATH=/path/to/.office-node/node_modules` if needed.
