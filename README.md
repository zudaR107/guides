# ZudaR Docs

**Pragmatic tech guides & theory.** Installations, tooling, workflows, troubleshooting — written in a no‑BS style, with screenshots and copy‑paste commands. Built with **MkDocs** + **Material**.

👉 Live site: **[https://docs.zudar.ru](https://docs.zudar.ru)**

---

## What’s inside

* Operating Systems: Arch Linux (bootable USB, Ventoy, archinstall, dual‑boot, btrfs, GRUB/UEFI)
* Coming soon: more practical **and** theoretical posts beyond OS — dev tooling, networking, automation, security, etc.
* Bilingual: **Russian (default)** and **English**.

---

## Quick start

```bash
# 1) Python env (recommended)
python -m venv .venv && source .venv/bin/activate

# 2) Install deps
pip install -U mkdocs mkdocs-material mkdocs-static-i18n

# 3) Run locally
mkdocs serve
# open http://127.0.0.1:8000/

# 4) Build static site
mkdocs build

# Optional: deploy to GitHub Pages
mkdocs gh-deploy
```

---

## Internationalization (i18n)

This repo uses **`mkdocs-static-i18n`** with the **suffix strategy** so language links always land on the correct page.

* Russian files live as `*.md` (default language)
* English translations live alongside as `*.en.md`

Example structure:

```
docs/
  index.md
  index.en.md
  os/
    bootable-usb.md
    bootable-usb.en.md
    ventoy.md
    ventoy.en.md
    arch-install.md
    arch-install.en.md
  images/...
```

Key config (already in `mkdocs.yml`):

```yaml
plugins:
  - i18n:
      docs_structure: suffix
      reconfigure_material: true
      languages:
        - locale: ru
          name: Русский
          default: true
          build: true
        - locale: en
          name: English
          build: true
```

---

## Writing guidelines

* **Keep it practical.** Short intros, clear steps, real commands.
* **Tone:** concise, a bit cheeky is fine, but stay respectful.
* **Structure:** `# Title` → short context → `## Plan / Steps` → screenshots → result/summary.
* **Code blocks:** prefer shell fenced blocks with language hints (`bash`, `powershell`, etc.).
* **Links:** point to upstream docs (Arch Wiki, official repos) where helpful.

---

## Contributing

PRs welcome:

1. Create a feature branch.
2. Add/modify Markdown under `docs/`.
3. Include screenshots under `docs/images/...` (PNG/JPG; keep sizes reasonable).
4. For English, add `*.en.md` siblings.
5. Run `mkdocs serve`, verify both languages.
6. Open a Pull Request.

---

## Repository layout

```
docs/           # source Markdown and images
mkdocs.yml      # site configuration
site/           # built static site (output)
```

> The `site/` folder is generated — keep it out of commits if you deploy elsewhere. For GitHub Pages via `gh-deploy`, it’s fine to let the action manage it.

---

## License

Such a resource should be completely **unlicensed**! And accessible to **everyone**.

---

## Contact

* Author: [@zudaR107](https://github.com/zudaR107)
* Telegram: [https://t.me/vrubovoy](https://t.me/vrubovoy)

Enjoy the guides! 🚀
