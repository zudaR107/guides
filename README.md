# ZudaR Docs

**Pragmatic tech guides & theory.** Installations, tooling, workflows, troubleshooting, security, and infrastructure — written in a no‑BS style, with screenshots and copy‑paste commands. Built with **MkDocs** + **Material for MkDocs**.

👉 Live site: **[https://docs.zudar.ru](https://docs.zudar.ru)**

---

## What’s inside

Current topics include:

* **Operating Systems** — bootable USBs, Ventoy, Arch Linux installation and setup.
* **Git and repos** — SSH, GPG, repository workflow, and related setup.
* **Bilingual content** — **Russian** is the default language, with **English** translations alongside it.

More practical and theoretical posts will be added over time: dev tooling, networking, automation, security, server setup, and other useful things.

---

## Quick start

```bash
# 1) Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate

# 2) Install dependencies
python -m pip install --upgrade pip
pip install -r requirements.txt

# 3) Run locally
mkdocs serve
# open http://127.0.0.1:8000/
# and also check http://127.0.0.1:8000/en/

# 4) Build the static site
mkdocs build --strict
```

> `mkdocs build --strict` is the recommended final check before pushing changes.

---

## Deployment

Deployment is handled automatically via **GitHub Actions**.

The workflow installs dependencies from `requirements.txt`, builds the site in strict mode, and publishes it to **GitHub Pages**.

Typical flow:

1. Make changes locally.
2. Run `mkdocs serve` and verify both languages.
3. Run `mkdocs build --strict`.
4. Commit and push to `main`.
5. GitHub Actions handles the rest.

If you keep drafts or unfinished posts, it is better to work in separate branches and merge only ready content into `main`.

---

## Internationalization (i18n)

This repo uses **`mkdocs-static-i18n`** with the **suffix** strategy.

That means:

* Russian files live as regular `*.md` files.
* English translations live alongside as `*.en.md` files.

Example:

```text
docs/
  index.md
  index.en.md
  os/
    arch-install.md
    arch-install.en.md
  git/
    ssh-gpg.md
    ssh-gpg.en.md
```

The site is configured so both languages are built and available in navigation.

---

## Writing guidelines

* **Keep it practical.** Short intro, clear steps, real commands.
* **Prefer structure.** Title → context → steps → screenshots → result.
* **Use fenced code blocks** with language hints: `bash`, `fish`, `powershell`, `yaml`, etc.
* **Prefer upstream links** when official documentation is useful.
* **If you add a Russian article, add an English version too** when possible.

---

## Contributing

PRs are welcome.

Basic flow:

1. Create a branch.
2. Add or update Markdown files under `docs/`.
3. Put screenshots into `docs/images/...`.
4. Keep the bilingual structure intact (`*.md` + `*.en.md` where applicable).
5. Run `mkdocs serve`.
6. Run `mkdocs build --strict`.
7. Open a Pull Request.

---

## Repository layout

```text
docs/             # source Markdown, translations, images, and CNAME
  git/            # Git / SSH / GPG guides
  os/             # OS-related guides
  images/         # screenshots and illustrations
mkdocs.yml        # MkDocs configuration
requirements.txt  # pinned Python dependencies
README.md         # this file
```

`site/` is a generated build artifact and should not be committed.

---

## Local notes

A few useful details:

* Python dependencies are installed strictly from `requirements.txt`.
* `.venv/`, `site/`, `.cache/`, and `__pycache__/` are ignored.
* The local dev server may occasionally show noisy warnings for paths like `sitemap.xml`; if the pages render correctly, that is usually not a real content issue.

---

## License

Such a resource should be completely **unlicensed** — and accessible to **everyone**.

---

## Contact

* Author: [@zudaR107](https://github.com/zudaR107)
* Telegram: [https://t.me/vrubovoy](https://t.me/vrubovoy)

Enjoy the guides! 🚀
