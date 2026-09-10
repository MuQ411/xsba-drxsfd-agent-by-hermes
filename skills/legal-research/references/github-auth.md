# GitHub Authentication for Skill Updates

This repo (`MuQ411/xsba-drxsfd-agent-by-hermes`) hosts the legal-research skill.
Pushing requires a working credential — check yours before promising a push.

## Quick credential audit (run this FIRST)

Before attempting any push, enumerate what actually works. Do not assume a
stored credential is valid just because the file exists.

```bash
# 1. gh CLI?
command -v gh && gh auth status

# 2. stored git credential — read it BYTE-LEVEL (see redaction trap below)
python3 - <<'EOF'
import re
raw = open("/home/mqy89/.git-credentials").read().strip()
m = re.match(r'https://([^:]+):(.+)@github\.com/?', raw)
if m:
    user, tok = m.group(1), m.group(2)
    kind = ("Fine-grained (github_pat_/github_)" if tok.startswith(("github_pat_","github_"))
            else "Classic (ghp_)" if tok.startswith("ghp_") else "unknown")
    print(f"user={user} len={len(tok)} kind={kind}")
else:
    print("no github.com credential stored")
EOF

# 3. does it actually authenticate? (read-only probe)
curl -s -H "Authorization: Bearer $TOK" https://api.github.com/user
#    -> 401 "Bad credentials" means DEAD. Do not proceed.
```

## ⚠️ The redaction trap: `***` may be the real value

Hermes redacts secrets in tool output. Reading a token via `read_file`,
`grep`, or any tool that prints to the conversation shows `***` — **which looks
exactly like a token that was written as the literal string `***`.**

These are different situations with different fixes:

| What you see | What it is | Action |
|---|---|---|
| `***` in output, but auth works | display-layer redaction | nothing — token is fine |
| `***` **in the raw bytes** | value is genuinely the 3-byte string `2a2a2a` | rewrite the value |

**The only way to tell them apart is to read the raw bytes and print their
length / hex — never the decoded string:**

```python
# Confirmed-good pattern: print length + hex, not the value
raw = open("/home/mqy89/.hermes/.env", "rb").read()
for i, line in enumerate(raw.split(b"\n"), 1):
    if b"=***" in line.split(b"=",1)[-1].strip():
        pass
    if b"GITHUB_PERSONAL_ACCESS_TOKEN" in line and not line.strip().startswith(b"#"):
        val = line.split(b"=", 1)[1]
        print(i, "bytes:", len(val), "hex:", val.hex())
# genuinely broken ->  bytes: 3  hex: 2a2a2a
# healthy secret  ->  bytes: 40+  hex: 3<...>
```

A healthy neighbouring secret is the control: if `.env` shows
`TELEGRAM_BOT_TOKEN` with 46 bytes of real hex while `GITHUB_PERSONAL_ACCESS_TOKEN`
shows 3 bytes (`2a2a2a`), redaction is *not* masking it — that line is corrupt.

## Why MCP GitHub reads work but writes fail

`GITHUB_PERSONAL_ACCESS_TOKEN` being corrupt/harvested-as-placeholder produces a
**confusing asymmetry**:

- `mcp__github__get_file_contents` on a **public** repo → succeeds (no auth needed)
- `mcp__github__create_or_update_file` → `McpError: Authentication Failed: Requires authentication`

Reads succeeding is **not** evidence the token is good. Always test a write path
(or `GET /user`) before concluding auth is healthy.

## Fine-grained PAT limitation (git push)

Fine-grained PATs (`github_pat_*` / newer `github_*`) **cannot** be used for
`git push` over HTTPS. GitHub returns:

```
remote: Invalid username or token.
Password authentication is not supported for Git operations.
```

They are intended for the REST API only, and even there they must carry
`Contents: Read and Write` on the specific repository.

## Correct approach: Classic PAT

Create at https://github.com/settings/tokens → **Generate new token (classic)**.

Required scope: `repo` (full control of repositories).

```bash
git push https://USERNAME:TOKEN@github.com/MuQ411/xsba-drxsfd-agent-by-hermes.git main
```

Or install `gh` and let it manage credentials:

```bash
sudo apt install gh
gh auth login            # or: echo "TOKEN" | gh auth login --with-token
gh auth setup-git
```

## Passing a fresh token without it being redacted

Because tool output is redacted, a token pasted into the conversation may be
mangled by the time it reaches a shell. Write it to a file with `write_file`
(the file write is not redacted the way stdout is), use it, then delete it:

```bash
# 1. write_file the token to /tmp/gh_token.txt
# 2. use it
git push "https://MuQ411:$(cat /tmp/gh_token.txt)@github.com/MuQ411/xsba-drxsfd-agent-by-hermes.git" main
# 3. clean up
rm -f /tmp/gh_token.txt
```

## Repo layout (as of the latest restructure)

```
README.md
skills/
└── legal-research/
    ├── SKILL.md                          # main skill body
    └── references/
        ├── companion-skills.md
        ├── drxsfd-bh-full-mapping.md
        ├── github-auth.md                # this file
        └── site-exploration-methodology.md
```

Earlier revisions stored the skill **flat** as `skills/legal-research.md`. If you
find that path, it is stale — the skill moved to the directory layout so the
`references/` payload could ship alongside it. Prefer `cp -r` of the whole
directory over copying a single file.

## API-only fallback

If only a token is available (no git), use the Contents API:

```python
PUT /repos/MuQ411/xsba-drxsfd-agent-by-hermes/contents/<path>
{
  "message": "commit message",
  "content": "<base64-encoded file content>",
  "branch": "main",
  "sha": "<existing blob sha, required when updating>"   # omit when creating
}
```

The `sha` of an existing file must be fetched first (`GET /contents/<path>`) or
the update is rejected. This is the sync point that makes a multi-file push
fiddly by hand — prefer git when the credential allows it.
