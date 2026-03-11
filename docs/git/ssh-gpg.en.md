# Git and repos. Part 1 — “SSH and GPG” 🔐

Once you start working with repositories properly, you pretty quickly run into two things: **SSH** and **GPG**. The first one lets you access remote repositories conveniently and securely without typing your login and password forever. The second one is for **signing commits and tags**, proving that yes, it was actually you — not some random dude named Bob from a suspicious basement.

In this article, we’ll calmly set both up without unnecessary drama: create an **SSH key** for **GitHub / Gitea / servers**, then generate a **GPG key** for signing commits. Along the way, we’ll also cover **what all this stuff actually is**, how to use it, and why private keys should be guarded like the crown jewels.

Let’s go 😎

---

## 🔑 What SSH is and why you need it

**SSH** (**Secure Shell**) is a secure protocol for remote access and authentication. Put simply, it lets you prove to the other side that you are who you say you are **without sending your password over the network every single time**.

In the Git world, this is especially nice. Instead of typing your login, password, tokens, and other misery every time you run `git clone`, `git pull`, or `git push`, you create a key pair once:

* a **private key** — kept only on your machine;
* a **public key** — uploaded to **GitHub**, **Gitea**, a server, or wherever needed.

Then the server goes: “Ah yes, I know this public key” — and lets you in. Lovely.

> **Important:** never give your private key to anyone. Ever. Seriously. A public key is meant to be shared and copied to servers. A private key is your digital pass.

---

## ⚙️ Why ed25519

For SSH keys, the sane default today is **ed25519**.

What even is that? It’s a modern algorithm based on **elliptic curves, specifically EdDSA over Curve25519**. Sounds like something muttered by a cryptographer in a dimly lit basement, but the practical part is what matters: **ed25519** is secure, fast, compact, and supported by basically every normal modern system.

Why it’s good:

* compact;
* fast;
* secure;
* supported almost everywhere.

If you’re not dealing with some ancient museum-grade infrastructure held together with duct tape and `rsa`, **ed25519** is the right choice.

---

## 🛠️ Creating an SSH key

Let’s generate one regular key for your local system:

```bash
ssh-keygen -t ed25519
```

Then the system will ask:

1. **Where to save the key.** You can keep the default path.
2. A **passphrase** for the key. I **strongly recommend** setting one.

If you keep the default path, you’ll end up with two files:

* `~/.ssh/id_ed25519` — private key;
* `~/.ssh/id_ed25519.pub` — public key.

---

## 🧠 Adding the SSH key to the agent

To avoid typing the passphrase every single time, you can load the key into **ssh-agent**:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

Yes, the `eval` command is most comfortable in **bash** or a compatible shell. But in general, this can be adapted just fine for other shells too.

My usual setup is simple: after the first use, the key gets loaded into the agent and stays there **until the terminal session is restarted**. Convenient, without going full clown mode.

If you use **fish**, it makes sense to add this to `~/.config/fish/config.fish`:

```fish
# Start ssh-agent if no agent socket is available.
if not set -q SSH_AUTH_SOCK
    eval (ssh-agent -c) > /dev/null
end
```

That way, the agent starts automatically if there’s no available socket for the current session yet. Small thing, nicer life.

---

## 📄 How to get your public SSH key

You can print the public part of the key directly in the terminal:

```bash
cat ~/.ssh/id_ed25519.pub
```

This is the text you copy:

* to **GitHub** → `Settings` → `SSH and GPG keys`;
* to **Gitea** → SSH key settings;
* to remote servers in `~/.ssh/authorized_keys`.

> Once again: the `.pub` file is the **public** key. That’s the one you share.

---

## 🧩 Configuring SSH

To avoid typing a pile of parameters by hand every time, edit `~/.ssh/config`.

Example:

```sshconfig
Host *
  ServerAliveInterval 30
  ServerAliveCountMax 3
  TCPKeepAlive yes
  AddKeysToAgent yes
  IdentitiesOnly yes
  HashKnownHosts yes
  StrictHostKeyChecking ask
  IdentityAgent $SSH_AUTH_SOCK

Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519

Host myvps
  HostName your.server.domain_or_ip
  User your_user
  Port 22
  IdentityFile ~/.ssh/id_ed25519
```

What’s useful here:

* **AddKeysToAgent yes** — the key can be added to the agent automatically;
* **IdentitiesOnly yes** — SSH won’t try every key in your personal key zoo;
* **HashKnownHosts yes** — the known hosts file is stored in hashed form;
* **StrictHostKeyChecking ask** — SSH will ask for confirmation on first connect.

After editing it, don’t forget to lock down permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
```

---

## ✅ Testing SSH with GitHub

Now you can test the connection:

```bash
ssh -T git@github.com
```

If everything is fine, GitHub will greet you and say authentication was successful. It won’t give you a shell though — that part is normal.

---

## 🖥️ SSH for your own server

GitHub is fairly civilized: paste the public key into settings and move on. Your own server is a bit more hands-on, but the logic is the same.

### 1. First login — with a password

To get into the server for the first time, password login **must be enabled**. Usually that is configured in `sshd_config` like this:

```conf
PasswordAuthentication yes
KbdInteractiveAuthentication yes
UsePAM yes
PermitRootLogin no
```

What those settings mean:

* **PasswordAuthentication yes** — allows classic password-based SSH login;
* **KbdInteractiveAuthentication yes** — enables keyboard-interactive authentication, which some systems still use together with password login;
* **UsePAM yes** — allows PAM to participate in authentication; on many systems this is part of normal password login behavior;
* **PermitRootLogin no** — root login over SSH is disabled. And that’s good. Logging in as `root` directly from the outside world is a bad habit and usually unnecessary.

There is one important catch here: your system may also have **additional SSH config files** that override these settings. Something like `sshd_config.d/*.conf`, for example. So you may set `PasswordAuthentication yes`, restart SSH, and the server still acts like it has never heard of the idea. Check not only the main config, but also the included config directory.

### 2. Copy the key to the server

After the first password login, copy your public key to the server:

```bash
ssh-copy-id your_user@your.server.domain_or_ip
```

If you use a non-standard port, specify it explicitly:

```bash
ssh-copy-id -p 2222 your_user@your.server.domain_or_ip
```

This command adds your public key to `~/.ssh/authorized_keys` on the server.

### 3. Now key-based login should work

After that, try a normal login:

```bash
ssh your_user@your.server.domain_or_ip
```

If everything is configured correctly, the server should let you in using the key. No password needed anymore — which is exactly the point.

### 4. Disable password login

And this part is **mandatory**: once you have confirmed that key-based login works, **disable password login**.

Your server config should end up like this:

```conf
PasswordAuthentication no
KbdInteractiveAuthentication no
UsePAM yes
PermitRootLogin no
```

Then restart `sshd` and sleep a little better.

Yes, this means that when you connect from a **new device**, you will no longer be able to just type a password and walk in like it’s 2009. You’ll need to:

1. either log in from an **old trusted device**;
2. or use a **web console / VNC / hosting panel**;
3. temporarily enable password login again;
4. add the new key;
5. and disable password login again.

Yes, it’s less convenient. **But that is security.** A server that allows password login from the outside world always has a larger attack surface. Better to set it up properly once than later discover a surprise crypto miner and a full traveling circus from the other side of the planet.

---

## 🔏 What GPG is and why you need it

Now for **GPG**.

**GPG** (**GNU Privacy Guard**) is a tool for cryptography: encryption, signature verification, and key management. In the Git context, we’re not interested in encryption here — we care about **digital signatures**.

Why sign commits:

* GitHub / Gitea can show that a commit is **signed and verified**;
* it becomes easier to prove that the commit was actually made by you;
* it’s just good practice if you maintain public repositories or serious projects.

If SSH says, “I’m allowed to access this server,” then GPG says, “I am the one who signed this commit.”

---

## 🛠️ Creating a GPG key

Generate a key in interactive mode:

```bash
gpg --full-generate-key
```

Recommended choices:

* **Key type** — `ECC (sign only)`;
* **Curve** — `Curve 25519`;
* **Expiration** — whatever you prefer. I usually set **1 year**;
* then enter your **name**, **email**, and **passphrase**.

Why one year is a reasonable choice:

* rotating keys from time to time is healthy;
* if a key leaks or becomes irrelevant, you won’t be stuck with it forever;
* a year is comfortable and not annoying.

---

## 🔍 Viewing your GPG key

After generating it, list your secret keys:

```bash
gpg --list-secret-keys --keyid-format=long
```

The output will look something like this:

```text
sec   ed25519/AAAAAAAAAAAAAAAA 2026-03-09 [SC] [expires: 2027-03-09]
      XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
uid                 [ultimate] Your Name <your_email@example.com>
```

What we care about here is the **key ID** — in this example, `AAAAAAAAAAAAAAAA`.

Now export the public part:

```bash
gpg --armor --export AAAAAAAAAAAAAAAA
```

That output should be pasted into:

* **GitHub** → `Settings` → `SSH and GPG keys` → `New GPG key`;
* or the equivalent section in **Gitea**, if your instance supports commit signature verification.

---

## 🧷 Configuring gpg-agent

To make passphrase prompts less annoying, configure **gpg-agent**. Edit `~/.gnupg/gpg-agent.conf`:

```conf
pinentry-program /usr/bin/pinentry-curses
default-cache-ttl 2147483647
max-cache-ttl 2147483647
```

What this means:

* **pinentry-program** — which program will ask for the passphrase;
* **default-cache-ttl** and **max-cache-ttl** — how long the agent keeps the passphrase in memory.

Yes, those values look like we’re trying to break math. In practice, this gives you a very long cache. Realistically, it often ends up behaving like “the passphrase stays cached **until you restart the terminal session or the agent**.”

If you work in the terminal, `pinentry-curses` is a good option. If you prefer graphical prompts, you can use `pinentry-gtk` or `pinentry-qt`.

If you use **fish**, add this to `~/.config/fish/config.fish` too:

```fish
# Ensure GPG pinentry can access the correct TTY.
set -gx GPG_TTY (tty)
```

Without this line, `pinentry` sometimes starts improvising and pretending it has no idea which terminal it should talk to. We don’t need that kind of creativity.

After editing the config, restart the agent:

```bash
gpgconf --kill gpg-agent
gpgconf --launch gpg-agent
```

And don’t forget proper permissions for the directory:

```bash
chmod 700 ~/.gnupg
```

---

## ⚙️ Connecting GPG to Git

Now tell Git which key to use for signing:

```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
git config --global user.signingkey AAAAAAAAAAAAAAAA
git config --global commit.gpgsign true
git config --global tag.gpgsign true
git config --global gpg.program gpg
```

What’s happening here:

* `user.name` and `user.email` — commit authorship;
* `user.signingkey` — which GPG key Git should use;
* `commit.gpgsign true` — sign all commits by default;
* `tag.gpgsign true` — sign tags;
* `gpg.program gpg` — which binary to use.

After that, Git will start signing commits automatically.

---

## ✅ Verifying commit signatures

The easiest way is to create a test commit:

```bash
mkdir test-gpg && cd test-gpg
git init
echo "test" > README.md
git add README.md
git commit -m "test: verify GPG signing"
```

If everything is configured correctly, Git will ask for your passphrase through `pinentry`, create the commit, and the platform will show that it is **signed**.

You can also inspect the signature locally:

```bash
git log --show-signature -1
```

---

## 🧠 How to store keys properly

The logic here is simple.

For your local system, create **one SSH key** and **one GPG key**. That is usually enough. Use the SSH key for GitHub, Gitea, and, if you want, your own servers. Use the GPG key for signing commits and tags.

The main rules:

* do not copy private keys all over the place;
* do not send them through messengers, cloud notes, or “temporary” storage;
* set a passphrase whenever possible;
* public keys can be added to different servers and services.

If you lose the device or suspect the key has leaked, the process is simple:

1. remove the public key from GitHub / Gitea / servers;
2. create a new pair;
3. update the bindings.

No panic — but also no “eh, it’ll probably be fine” mode.

---

## 🏁 What we ended up with

We configured two important mechanisms:

* **SSH** — for secure access to GitHub / Gitea / servers;
* **GPG** — for signing commits and tags.

Now you have a solid base for working with repositories: no endless password typing, cleaner authentication, and verifiable commits. No longer just “well, something got pushed somehow,” but a somewhat grown-up setup 🙂

And the main thing now is simple: don’t leak your private keys all over the place, and don’t build a digital junkyard where one clean setup would do just fine 🖤
