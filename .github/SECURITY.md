# Security Policy

This profile is owned by **Yassir Zahidi** — cybersecurity engineering student, builder of [`home-lab-siem`](https://github.com/y-zahidi/home-lab-siem). I take responsible disclosure seriously across every public repo on this account.

## Reporting a vulnerability

If you find a security issue in any repository under [`github.com/y-zahidi`](https://github.com/y-zahidi):

1. **Do not** open a public issue or pull request.
2. Email **yassirzahidi8@gmail.com** with subject line `security: <repo-name>`.
3. Include:
   - the affected repo + commit SHA / tag,
   - a minimal reproduction (PoC, payload, or steps),
   - your assessment of impact (confidentiality / integrity / availability),
   - whether you'd like to be credited.

I'll acknowledge within **48 h** and aim to ship a fix or mitigation within **7 days** for high-severity issues, **14 days** for medium, **30 days** for low. If a coordinated disclosure timeline works better for both sides, just say so in the first email.

## In scope

- All source code in repositories owned by `y-zahidi`.
- The portfolio site at [`y-zahidi.github.io`](https://y-zahidi.github.io) (static).
- Configuration files, CI workflows, container definitions in those repos.

## Out of scope

- Third-party services I link to (LinkedIn, Vercel, etc.) — report to the vendor.
- Social-engineering reports against the GitHub account itself — report to GitHub Security.
- Issues that require physical access to my devices.

## Safe harbor

Good-faith research that follows this policy will not be pursued legally. Don't exfiltrate data, don't pivot, don't disrupt service — keep it minimal and demonstrate impact with the smallest possible footprint.

## Hall of fame

Researchers who report a valid issue will be listed here (with permission, by handle). The list is empty for now — be the first.

---

*Policy version: 1.0 · last reviewed: 2026-05-10*
