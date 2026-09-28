+++
title = "Your Own Claude Code Cloud: Remote Control, Worktrees, and a Locked-Down Unix Account"
author = ["Rod Morison"]
date = 2026-09-27T10:45:00-07:00
tags = ["ai", "claude-code", "security"]
draft = false
topics = ["AI", "Claude", "Security"]
description = "How to run several independent Claude Code agents per repo on a server you control: a dedicated unix account, an egress firewall, systemd-managed Remote Control, git worktrees, plugins, and signed commits. An open-ended alternative to cloud sessions."
+++

<!-- hand-authored Hugo markdown — NOT an ox-hugo export from org/all-posts.org.
     Do not create an org entry with EXPORT_FILE_NAME "claude-code-own-cloud"
     or ox-hugo will overwrite this file. -->

{{< figure src="/ox-hugo/claude-code-own-cloud-cumulus.jpg" alt="Towering storm clouds, dark on the left and sunlit white on the right, with a patch of deep blue sky between them" >}}

## Why Bother? {#why-bother}

Claude Code's cloud sessions are great for a quick task against a GitHub repo: pick the repo, type a prompt, get a PR. But after a few weeks of real work in them I kept hitting the same walls:

- **Each session is its own container.** Fresh clone, fresh environment, reclaimed when idle. Nothing I install or configure sticks.
- **One top-level agent per container.** Subagents, yes. But I wanted several *independent* agents on the same repo at once, each on its own branch, each one I can talk to directly.
- **The environment is theirs, not mine.** Tooling, plugins, network policy and credentials are whatever the environment allows.

Meanwhile I already run a dedicated Linux server. Claude Code's **Remote Control** mode lets a process on your own machine show up in claude.ai and the mobile app as a place to run sessions. Add git worktrees, a locked-down unix account, and systemd, and you get something that feels like the cloud product but is yours: persistent, extensible, and as open or closed as you choose to make it.

This is the how-to.

## What You End Up With {#what-you-end-up-with}

<a href="/ox-hugo/claude-code-own-cloud-architecture.svg" target="_blank" rel="noopener"><img src="/ox-hugo/claude-code-own-cloud-architecture.svg" style="display:block; margin:1em auto; width:100%;" alt="Architecture: your devices talk to Anthropic, and your server connects out to Anthropic over HTTPS with no inbound ports. On the server, a locked-down ai-dev account runs one Remote Control service per repo, each running up to four agents in their own git worktrees, sharing user-scope plugins and persistent clones. A per-user nftables policy allows only HTTPS, DNS and SSH to GitHub, and blocks the server&#39;s own mail, database and cache services." /></a>
<p style="text-align:center; font-size:0.85em; margin-top:-0.5em;"><em>Click the diagram to open it full size.</em></p>

In the claude.ai session picker, each repo shows up under **Remote Control**. Click **New**, pick the repo, type a prompt, and you've started another top-level agent in its own worktree.

**You'll need:** a Linux box with systemd (I'm on Ubuntu 22.04), sudo on it, a Claude Pro or Max subscription, and a GitHub account.

### Or Let Claude Code Set It Up {#let-claude-code-set-it-up}

I didn't type most of this. A Claude Code session on the same server drafted every command, and I ran the root ones. You can do the same. Point a session with SSH and sudo at this page, but keep it on a short leash:

> Read https://rm.rmdashrf.net/posts/claude-code-own-cloud/ and set this up on this server for a user called `ai-dev` and the repo `<owner>/<repo>`. Show me each sudo command and wait for my OK before running it. Stop and hand me the terminal for the Claude login, the GitHub token, and the trust prompt. Run the firewall tests at the end and show me the raw output.

Four steps stay yours: the Claude `/login`, creating and pasting the GitHub token, adding the signing key on GitHub, and the "trust this folder" prompt. Secrets should never pass through an agent's conversation.

**Caveat emptor:** the agent doing the setup runs *outside* the protections it's building, as your admin account, with root. So don't run it in auto mode: approve each command. This is also exactly where **prompt injection** bites. A page that feeds an agent root commands is effectively running your server, and a copied, tampered-with or outright malicious version of these instructions could slip in one extra line you'd never notice.

<ul style="list-style:none; padding-left:0.25em;">
<li style="padding-left:0;"><span style="color:#3b82f6; font-weight:700; margin-right:0.5em;">➜</span><strong>Stick to the canonical URL.</strong></li>
<li style="padding-left:0;"><span style="color:#3b82f6; font-weight:700; margin-right:0.5em;">➜</span><strong>Read every command before you approve it.</strong></li>
<li style="padding-left:0;"><span style="color:#3b82f6; font-weight:700; margin-right:0.5em;">➜</span><strong>Read the test output yourself</strong> rather than taking "all green" on faith.</li>
</ul>

## 1. A Dedicated Account {#a-dedicated-account}

Agents run shell commands, and in auto mode they don't ask first. So they don't run as me. My convention is one `<name>-dev` account per separately threaded body of work. This one is `ai-dev`.

```bash
sudo adduser --disabled-password --gecos "AI dev agents" ai-dev
sudo chmod 750 /home/ai-dev
sudo loginctl enable-linger ai-dev
sudo systemctl set-property user-$(id -u ai-dev).slice MemoryMax=12G CPUQuota=500%
```

- **No password, no sudo, no groups.** Watch out for `docker` in particular: membership in it is root in all but name.
- **No SSH keys either.** I get in from my own account with `sudo -iu ai-dev`. Remote Control only makes outbound connections, so the account never needs to accept a login.
- **Linger** keeps ai-dev's services running across reboots and logouts. By default, Linux ties a user's background services to their login: they start when you log in, stop when you log out, and don't come back after a reboot until you log in again. Fine for a desktop, but ai-dev never logs in; you just borrow its shell with `sudo`. Linger tells systemd to start its services at boot and keep them running regardless, so the Remote Control servers in step 6 behave like always-on server software.
- **The slice caps** memory and CPU (5 of 8 cores here), so a runaway test suite can't starve everything else on the box.

## 2. An Egress Firewall for That Account Only {#an-egress-firewall}

This step matters more than it looks. A typical server runs services on localhost that trust *any local user*: an MTA that relays mail from 127.0.0.1, a Redis or memcached with no password, and so on. A hijacked agent that can talk to your MTA can send mail as you, from your IP.

nftables (the successor to iptables; on recent Ubuntu, `iptables` is already a front end to it) can match packets on the **uid of the process that sent them**. The table below applies only to ai-dev and leaves every other user alone. It's a separate table, so it coexists with ufw. A packet has to pass both, and a drop in either wins.

**Create the rules file.** Adjust the `local_svcs` ports to match what your box listens on (check with `ss -ltnp`), then run this as a sudoer to write `/etc/nftables-ai-dev.nft`:

```bash
sudo tee /etc/nftables-ai-dev.nft >/dev/null <<'EOF'
table inet ai_dev {}
delete table inet ai_dev
table inet ai_dev {
  set local_svcs {
    type inet_service
    elements = { 25, 465, 587, 5432, 6379, 11211 }   # mail, postgres, redis, memcached: list yours
  }
  # GitHub git endpoints (from api.github.com/meta, "git" key)
  set github_git4 {
    type ipv4_addr; flags interval
    elements = { 140.82.112.0/20, 143.55.64.0/20, 185.199.108.0/22, 192.30.252.0/22 }
  }
  set github_git6 {
    type ipv6_addr; flags interval
    elements = { 2a0a:a440::/29, 2606:50c0::/32 }
  }
  chain out {
    type filter hook output priority 0; policy accept;
    meta skuid != "ai-dev" accept               # everyone else: untouched
    ct state established,related accept
    oif "lo" tcp dport @local_svcs drop         # no local mail/db/cache
    oif "lo" accept                             # DNS stub + its own dev servers
    tcp dport @local_svcs drop                  # ...nor via the public IP
    tcp dport { 80, 443 } accept
    udp dport { 53, 443 } accept
    tcp dport 53 accept
    ip  daddr @github_git4 tcp dport 22 accept  # ssh to GitHub only
    ip6 daddr @github_git6 tcp dport 22 accept
    drop
  }
}
EOF
```

The `table … {}` / `delete table …` pair at the top is a clean-reload idiom for older nft versions without `destroy`: reloading the file always gives you exactly its contents, never duplicate rules.

**Load it, persist it with a small oneshot unit, and test it.** Don't put the rules in `/etc/nftables.conf`: that file starts with `flush ruleset`, which would wipe ufw's rules too.

```bash
sudo nft -c -f /etc/nftables-ai-dev.nft && sudo nft -f /etc/nftables-ai-dev.nft

sudo tee /etc/systemd/system/nft-ai-dev.service >/dev/null <<'EOF'
[Unit]
Description=nftables egress policy for ai-dev
After=network-pre.target ufw.service
[Service]
Type=oneshot
ExecStart=/usr/sbin/nft -f /etc/nftables-ai-dev.nft
RemainAfterExit=yes
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload && sudo systemctl enable --now nft-ai-dev

# Expect: BLOCKED · an HTTP status line · OPEN · BLOCKED
sudo -u ai-dev timeout 5 bash -c 'exec 3<>/dev/tcp/127.0.0.1/25' && echo OPEN || echo BLOCKED
sudo -u ai-dev curl -sI https://api.anthropic.com | head -1
sudo -u ai-dev timeout 5 bash -c 'exec 3<>/dev/tcp/github.com/22' && echo OPEN || echo BLOCKED
sudo -u ai-dev timeout 5 bash -c 'exec 3<>/dev/tcp/gitlab.com/22' && echo OPEN || echo BLOCKED
```

**Be honest with yourself about what this does and doesn't buy.** With 443 open to the world, an agent can still upload anything it can read to any website. The firewall protects *your other services* and your IP's reputation. What protects your *data* is the account boundary: ai-dev simply can't read your home directory, your secrets or your mail. (Mail, Drive and friends come with one big asterisk: your Claude login's connectors, which [step 4](#cut-the-logins-reach) turns off.) The secrets it *does* hold are covered in [step 9](#secrets).

## 3. Install Claude Code and Log In {#install-claude-code}

```bash
sudo -iu ai-dev
```

Two gotchas before you paste anything else:

1. **`sudo -iu` starts a new shell, and that shell swallows the rest of a multi-line paste.** Run it on its own line, then paste the next block.
2. **`sudo -iu` doesn't create a systemd login session,** so `systemctl --user` fails with "Failed to connect to bus." Linger keeps the user manager running; the shell just needs to be told where it is.

```bash
cat >> ~/.bashrc <<'EOF'
export PATH="$HOME/.local/bin:$PATH"
export XDG_RUNTIME_DIR=/run/user/$(id -u)
EOF
source ~/.bashrc
curl -fsSL https://claude.ai/install.sh | bash
claude
```

Inside `claude`, run `/login` and pick the **Claude account with subscription**. On a headless box it prints a URL: open it on your laptop, sign in, and paste the code back.

Every agent here draws on your one subscription's usage limits. That's worth knowing before you start several at once.

## 4. Cut the Login's Reach {#cut-the-logins-reach}

Hat tip to u/bcRIPster, whose [PSA on r/ClaudeAI](https://www.reddit.com/r/ClaudeAI/comments/1wrr1z1/psa_for_anyone_using_claude_projects_to/) about claude.ai Projects quietly sharing account memory got me asking what else a Claude login carries along. The answer for this setup: a lot. Your Claude login isn't just a key to your subscription. It brings account-level features with it that go straight around the unix account and the firewall.

### Connectors {#turn-off-connectors}

Logging in with your claude.ai account brings the account's **connectors** with it. Whatever you've connected on claude.ai (Gmail, Google Drive, Calendar, Slack, Dropbox) shows up as tools in every Claude Code session on that login, on any machine. Run `claude mcp list` as ai-dev and look. I found them, all connected. The unix account boundary doesn't touch them, because the access rides on the Claude login, not on files. The firewall doesn't touch them either: it's all HTTPS.

So an agent in auto mode, one prompt injection away from a malicious web page or issue, could read your mail, post to Slack *as you*, or trash files in Drive. The fix is one setting that applies only to this unix account. The connectors keep working everywhere else you use Claude.

**As ai-dev, turn the claude.ai connectors off and confirm they're gone:**

```bash
f=~/.claude/settings.json; [ -f "$f" ] || echo '{}' > "$f"
jq '.env.ENABLE_CLAUDEAI_MCP_SERVERS = "false"' "$f" > "$f.tmp" && mv "$f.tmp" "$f"
claude mcp list        # expect no "claude.ai ..." entries
```

The service unit in step 6 sets the same variable, so the Remote Control sessions honor it too. For added peace of mind, add a `permissions.deny` entry for each connector's tool prefix (for example `mcp__claude_ai_Gmail`), taking the names from what `claude mcp list` showed before you turned them off.

### Cross-Session Messaging {#cross-session-messaging}

The login shares more than connectors. Plugins attached to your account sync in, but they run inside the sandbox. The one to watch is **cross-session messaging**: Claude Code sessions on the same login can list and message each other, across machines and unix users. By default, a message from a session in the same permission class (auto and ordinary prompting count as one) goes straight into the receiving agent's conversation, with no approval. A hijacked agent here could message your other sessions, and they could message it.

Close it from both ends with the `crossSessionInbound` setting. **As ai-dev, refuse inbound messages entirely:**

```bash
f=~/.claude/settings.json
jq '.crossSessionInbound = "refuse"' "$f" > "$f.tmp" && mv "$f.tmp" "$f"
```

**On machines where you run your own, more privileged sessions, hold inbound messages for your review.** Run the same command with `"hold"` in that machine's `~/.claude/settings.json`.

I tested both. Sends to the agent account now fail outright ("can't receive cross-session messages"), and a message to a session on my laptop was parked for approval instead of reaching Claude. Both took effect on sessions that were already running.

## 5. GitHub Access, Scoped Down {#github-access}

**Create a fine-grained personal access token** for ai-dev, not your everyday credentials. On GitHub: Settings → Developer settings → Fine-grained tokens → Generate new token:

- **Resource owner:** you. **Expiration:** 90 days.
- **Repository access:** *Only select repositories*.
- **Permissions:** Contents, Pull requests and Issues set to read/write. Actions and Commit statuses set to read. Add Workflows only if agents should edit CI.

Adding a repo later means editing the token's repo list on GitHub; nothing changes on the box.

**Then, as ai-dev, log `gh` in with the token and clone your first repo:**

```bash
gh auth login --hostname github.com --git-protocol https   # "Paste an authentication token"
gh auth setup-git
git config --global user.name  "Your Name"
git config --global user.email "you@users.noreply.github.com"
mkdir -p ~/projects/github.com/<owner> && cd ~/projects/github.com/<owner>
gh repo clone <owner>/<repo>
```

Pasting the token at `gh`'s prompt keeps it out of your shell history and process list. The `~/projects/github.com/<owner>/<repo>` layout is borrowed from Go's module paths: the path tells you the remote, and the remote tells you the path.

## 6. One Remote Control Service per Repo {#remote-control-service}

A systemd **template unit** gives you one Remote Control server per repo, each named after it.

**Create the unit file.** As ai-dev, replace `<owner>` with your GitHub user or org, then run:

```bash
mkdir -p ~/.config/systemd/user
cat > ~/.config/systemd/user/claude-rc@.service <<'EOF'
[Unit]
Description=Claude Code Remote Control (ai-dev: %i)
After=network-online.target
[Service]
WorkingDirectory=%h/projects/github.com/<owner>/%i
ExecStart=%h/.local/bin/claude remote-control --name ai-dev-%i --spawn worktree --capacity 4 --permission-mode auto
Restart=always
RestartSec=10s
Environment=PATH=%h/.local/bin:/snap/bin:/usr/local/bin:/usr/bin:/bin
Environment=ENABLE_CLAUDEAI_MCP_SERVERS=false
[Install]
WantedBy=default.target
EOF
```

The `%i` in the unit is the part after the `@`: enabling `claude-rc@myrepo` runs a server for `~/projects/github.com/<owner>/myrepo`. The three flags that make this work:

- **`--spawn worktree`** gives every session started from claude.ai its own git worktree and branch, so concurrent agents never step on each other's files. The other modes are `same-dir` (the default) and `session` (a single classic session).
- **`--capacity 4`** caps concurrent sessions per repo. The default is 32, which is more agents than your usage limits will feed.
- **`--permission-mode auto`** starts spawned sessions in auto mode instead of asking before every command. That's reasonable *because* of steps 1, 2 and 4; I wouldn't do it on my own account.

**Before enabling the service, do two one-time steps from inside the repo.** Claude Code won't save "trust this folder" for your home directory, by design, and the service fails in a restart loop ("Workspace not trusted") until the repo itself is trusted:

```bash
cd ~/projects/github.com/<owner>/<repo>
claude                                        # choose "Yes, I trust this folder", then /exit
echo y | timeout 8 claude remote-control --name ai-dev-<repo>   # one-time "Enable Remote Control?"
```

**Then reload systemd, enable the service for the repo, and watch it start:**

```bash
systemctl --user daemon-reload
systemctl --user enable --now claude-rc@<repo>
journalctl --user -u claude-rc@<repo> -f
```

To make auto mode the default for sessions you start by hand in an ai-dev shell too:

```bash
f=~/.claude/settings.json; [ -f "$f" ] || echo '{}' > "$f"
jq '.permissions.defaultMode = "auto"' "$f" > "$f.tmp" && mv "$f.tmp" "$f"
```

## 7. Plugins, Installed Once {#plugins}

Plugins installed at **user scope** apply to every session the account starts, in every repo. I use Every's [Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin) plugin: a brainstorm → plan → build → review → capture-learnings loop.

**As ai-dev, install it and restart the services:**

```bash
claude plugin marketplace add EveryInc/compound-engineering-plugin && \
claude plugin install compound-engineering@compound-engineering-plugin --scope user && \
systemctl --user restart 'claude-rc@*'
```

Sessions that were already open before the restart won't see new plugins; new ones will. Plugins attached to your claude.ai account also sync onto the box, and `claude plugin list` shows which copy wins when the names collide.

## 8. Signed Commits {#signed-commits}

Cloud sessions produce "Verified" commits. Yours can too, with an SSH signing key.

**As ai-dev, generate the key and turn on signing.** Use the same noreply address your commits use:

```bash
ssh-keygen -t ed25519 -C "ai-dev commit signing" -f ~/.ssh/id_ed25519_signing -N "" && \
git config --global gpg.format ssh && \
git config --global user.signingkey ~/.ssh/id_ed25519_signing.pub && \
git config --global commit.gpgsign true && \
git config --global tag.gpgsign true && \
echo "you@users.noreply.github.com $(cat ~/.ssh/id_ed25519_signing.pub)" > ~/.ssh/allowed_signers && \
git config --global gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers && \
cat ~/.ssh/id_ed25519_signing.pub
```

**Then register the printed public key on GitHub:** Settings → SSH and GPG keys → **New SSH key** (not the GPG form), set **Key type: Signing Key**, paste the whole line, and save. A signing key can sign commits but can't log in or push. There's no passphrase because unattended agents can't type one; if the box is ever compromised, delete the key on GitHub and it's dead. Git reads its config on every run, so already-running sessions pick this up with no restart.

## 9. Secrets: What the Account Holds {#secrets}

When you're done, ai-dev holds exactly three credentials, and one of them is bigger than it looks. All are plain files readable only by the account (mode 600); a headless box has no keyring to put them in.

| Secret | Where it lives | If it leaks, someone can… | Risk | Revoke |
|---|---|---|---|---|
| Claude login | `~/.claude/.credentials.json` | Run Claude Code on your subscription and burn your usage. With connectors left on, also reach your mail, Drive, Slack and more ([step 4](#cut-the-logins-reach)) | Medium-high; **high** if connectors are on | `/logout`, or end the session in claude.ai settings |
| GitHub token | `~/.config/gh/hosts.yml` | Push, open PRs and edit issues in the *selected repos only*, until it expires | Medium; low if `main` requires PRs | Delete the token on GitHub |
| Signing key | `~/.ssh/id_ed25519_signing` | Make "Verified" commits in your name. It can't push or log in | Low-medium | Delete the signing key on GitHub |

**The agents can read all of these.** They run as the same user, so there's no hiding a secret from them, and a malicious instruction buried in a web page or an issue could send one out over HTTPS. So the rule isn't "hide them well". It's **few, narrowly scoped, short-lived, and easy to revoke**.

**Do these:**

- **Protect `main`.** On GitHub, go to the repo's Settings → Branches and require a pull request before merging. A leaked token then gets "open a PR you'll review" instead of "push to production". It's one setting, and the best return in this table.
- **Keep app secrets out of the account.** No `.env` files full of production keys in the clones or worktrees. If tests need credentials, use dev-only keys with low limits, or none. (I wrote this up as a standard of its own: [keep application secrets out of the developer's working tree](https://github.com/rmorison/engineering-standards/pull/46).)
- **Know your revocation drill.** If the box is ever compromised, work down the table: log out Claude, delete the GitHub token, delete the signing key. It takes about five minutes.

What's deliberately *not* in the account matters just as much: no personal cloud keys, no mail, no connectors, nothing from your own account. That absence is the real protection.

## Using It {#using-it}

In claude.ai (or the desktop app, or the phone), start a new session, open the environment picker, and go to **Remote Control**. Each repo is listed with a count like *1 of 4 sessions*. Pick one and type. Every new session is another independent top-level agent in its own worktree.

It's just as usable from a phone. Here's one of those agents from my pocket: connected to the server, working an issue in its own worktree, and waiting on a one-letter answer before it carries on.

<img src="/ox-hugo/claude-code-own-cloud-phone-session.png" style="display:block; margin:1em auto; max-width:540px; width:100%; border:1px solid #cbd5e1; border-radius:12px;" alt="Claude mobile app session card titled &#39;#47 narrow uv allow entries plan&#39;, showing a green Connected status for the rmorison/engineering-standards repo and a message asking the user to reply A or B" />

A couple of things I learned the hard way:

- **The picker shows folder names, not your `--name`.** If two services run in folders with the same name (say, two users' clones of the same repo), you'll see duplicates. The capacity number is how I tell them apart.
- **Archiving a session frees its slot. The worktree and branch may stay behind**, which is a sensible safety default. Merge the PR, ask the agent to remove its worktree, then archive. `git worktree list` and `git worktree prune` handle the rest.

## Cloud vs. Your Own Box {#cloud-vs-your-own-box}

|                           | Claude Code cloud sessions  | Your own box                                 |
|---------------------------|-----------------------------|----------------------------------------------|
| Setup                     | none                        | an afternoon                                 |
| State between sessions    | fresh container each time   | persistent: tools, caches, clones            |
| Concurrent agents per repo| one per container           | `--capacity` per repo, each in a worktree    |
| Plugins and config        | what the environment allows | anything, installed once at user scope       |
| Network policy            | the environment's           | yours, down to per-uid firewall rules        |
| Hardware                  | theirs, fast                | yours, as old or new as it is                |
| Blast radius              | a disposable container      | whatever you let the account touch           |

The last row is the whole game. The cloud gives you isolation for free. On your own box, isolation is something you build, which is what steps 1, 2 and 4 are for. Do those first.

## Wrap-up {#wrap-up}

Put together, this is a small private "agent cloud": several independent agents per repo, reachable from a phone, with persistent tooling and a security boundary I can reason about line by line. And adding a repo is a clone, a trust prompt, and one `systemctl --user enable --now claude-rc@<repo>`.

It isn't a replacement for the cloud product so much as the open-ended version of it. When I want a quick, disposable session, I still use theirs. When I want a team of agents on a repo I care about, they run here.

{{< figure src="/ox-hugo/claude-code-own-cloud-keyboard-bench.jpg" alt="A piano keyboard under restoration on a scarred wooden workbench, hand tools, punches and tape rolls hanging on the wall behind it" >}}

**P.S.** I don't cover Docker or Podman here, but they're the obvious next step: agents will want test databases and `docker compose up`. Don't solve it by adding the account to the `docker` group. That's root-equivalent, and container traffic routes around the per-uid firewall. A rootless Docker daemon (or rootless Podman) owned by the agent account keeps both boundaries intact. That's a post of its own.

---

<p style="font-size:0.85em; color:#666; margin-top:2em;">Written with the editorial assistance of <a href="https://claude.ai" target="_blank" rel="noopener">Claude</a>. Drafted by Opus via <a href="https://www.claude.com/claude-code" target="_blank" rel="noopener">Claude Code</a>, from the session that did the setup described here.</p>
