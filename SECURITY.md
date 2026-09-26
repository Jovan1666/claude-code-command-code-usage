# Security Policy

## Supported versions

The latest commit on `main` is supported. Fixes land there; there are no backport
branches and no maintained older releases.

## Reporting a vulnerability

Report privately through GitHub: open the **Security** tab of
<https://github.com/Jovan1666/claude-code-command-code-usage> and choose **Report a
vulnerability**. If that channel is not available to you, open a normal issue that
says only that you have a security report and how to reach you — put no details in
the issue itself.

Say what you ran, what happened, and what you expected; a minimal reproduction is
worth more than a long description. This is a personal project maintained in spare
time, so expect an acknowledgement within a few days, and please hold public
disclosure until a fix is out.

## What this repository does with your machine

The shape of it *is* the security model:

| | |
|---|---|
| **Reads** | `~/.claude/settings.json` (the `env` block, to learn which local `ANTHROPIC_DEFAULT_*_MODEL` alias maps to which upstream model) and the session transcript the status line names on stdin — the transcript is opened only to read model names, and nothing from it is sent anywhere. |
| **Writes** | `~/.commandcode-usage/models.json`, the 24 h cache of the public model catalog — plus the host config change described below, which only `scripts/setup.mjs` makes. |
| **Sends** | HTTPS to `https://api.commandcode.ai` with **your own** key. No other host appears in the code. |
| **Collects** | Nothing. No telemetry, no analytics, no error reporting, no identifiers. |
| **Install** | The plugin itself runs nothing. No `postinstall` script, no downloaded code, no remote configuration. |

The API calls are `GET /alpha/whoami`, `/alpha/billing/credits`,
`/alpha/billing/subscriptions`, `/alpha/usage/summary` — all with your key — and
`/provider/v1/models`, which is public and needs no key at all.

### The one thing that writes host config

Claude Code has no plugin mechanism that can declare a status line, so the last
install step edits the host's config directly — and it does not do it silently.
`scripts/setup.mjs` sets the `statusLine` key in `~/.claude/settings.json`, after
writing a timestamped `.bak-*` copy of the file. It prints the exact path and the
diff it is about to write, accepts `--print` to show that without touching anything,
refuses to overwrite a status line written by someone else unless you pass `--force`,
and `--remove` puts your own status line back.

It also runs as the `statusLine` command, so the renderer executes once per assistant
message. A command that runs on every message is a command whose startup cost is a
feature; it is also a command whose file access should be this boring. Keep it that way.

## Credentials

Your Command Code key is discovered in this order, and used for nothing except the
`Authorization` header of the requests listed above:

1. `COMMAND_CODE_API_KEY`, `COMMANDCODE_API_KEY`, or `CMD_API_KEY`
2. any environment variable whose name contains `commandcode`
3. `~/.commandcode/auth.json` (written by the Command Code CLI login)
4. a Command Code provider route in `~/.claude/settings.json`

A key is never written to a log, to the cache, or into anything the plugin renders.
Prefer the environment variable over a literal in a config file that might get
committed. If you believe a key of yours is exposed, revoke it in your Command Code
account first — that is the only step that actually helps. (Failures do get logged —
`~/.claude/commandcode-statusline.log`, so a silently missing status line can be
explained — but only exit codes and the first line of stderr, never a credential.)

## Scope

In scope: anything in this repository that leaks a credential, sends data anywhere
other than the API base above, writes outside the paths listed here, or turns an
untrusted input (a session file, a transcript, a config value) into code execution.

Out of scope: the Command Code API itself; Claude Code and its plugin mechanism; and
anything that requires an attacker who already holds your key or your shell.
