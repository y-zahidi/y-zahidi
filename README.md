<!--
  ────────────────────────────────────────────────────────────────────────────
   yassir zahidi · cybersecurity engineering student · morocco
   i build a soc-grade lab on my own time, then i break it on purpose.
   every visual on this page is hand-coded svg, checked into the repo.
   no widgets, no shields-from-shields, nothing fetched at render time.
  ────────────────────────────────────────────────────────────────────────────
-->

<a href="https://y-zahidi.github.io">
  <img src="banner.svg" alt="Yassir Zahidi — cybersecurity engineering student · Morocco · blue · red · purple" />
</a>

<p align="center">
  <a href="https://y-zahidi.github.io"><img alt="portfolio" src="https://img.shields.io/badge/portfolio-y--zahidi.github.io-5cf2c1?style=flat-square&labelColor=04070d"/></a>
  <a href="https://github.com/y-zahidi/home-lab-siem"><img alt="home-lab-siem" src="https://img.shields.io/badge/home--lab--siem-docker_compose_up-79e2ff?style=flat-square&labelColor=04070d"/></a>
  <a href="https://www.linkedin.com/in/yassir-zahidi/"><img alt="linkedin" src="https://img.shields.io/badge/linkedin-yassir--zahidi-79e2ff?style=flat-square&labelColor=04070d"/></a>
  <a href="mailto:yassirzahidi8@gmail.com"><img alt="mail" src="https://img.shields.io/badge/mail-yassirzahidi8%40gmail.com-cfd6e4?style=flat-square&labelColor=04070d"/></a>
  <img alt="open" src="https://img.shields.io/badge/available-open-5cf2c1?style=flat-square&labelColor=04070d"/>
  <img alt="location" src="https://img.shields.io/badge/morocco-eu_%C2%B7_remote-79e2ff?style=flat-square&labelColor=04070d"/>
</p>

<h1 align="center">yassir zahidi</h1>
<p align="center"><i>cybersecurity engineering student · i build a soc-grade lab on my own time, then i attack it on purpose so it gets better.</i></p>

<img src="assets/cardstack.svg" alt="Three discipline cards — BLUE (defense — Wazuh, Suricata, Sysmon, MISP), RED (offense — nmap, BloodHound, Burp, Metasploit), PURPLE (validation loop — atomic-red-team, sigma, caldera)" />

> blue is what i train for. red is the gym. purple is the loop where the two meet — every red find becomes a new blue rule the next day, and the rule gets re-fired against the same payload until it sticks. the rest of this page is the receipts.

---

### `// table of contents`

```text
01  the-lab            home-lab-siem · architecture · proof
02  detections         triple-syntax sample · coverage matrix · playbook
03  evidence           one-year activity calendar · live console mock
04  history            préfecture de tétouan · alten · projects
05  toolchain          stack heatmap · certifications · principles
06  reach              four channels, async-first
```

---

## 01 · the lab — [`home-lab-siem`](https://github.com/y-zahidi/home-lab-siem)

A reproducible SOC-grade segment i run for myself. **Wazuh + Suricata + Sysmon + MISP + VirusTotal** behind a **FortiGate**, validated weekly with **Nessus** and beaten on regularly with **atomic-red-team**. The architecture mirrors what i helped deploy at the **Préfecture de Tétouan (SSIC, Ministère de l'Intérieur)** in May 2024 — repackaged so anyone can `docker compose up`.

I built the lab to learn defense end-to-end. Then i started attacking it on purpose to learn offense end-to-end. The two columns of this page are the same project, just from opposite sides of the firewall.

### `play /soc/killchain.mov`

A 30-second loop. An adversary walks every stage of the kill-chain — **recon → initial-access → foothold → priv-esc → lateral → exfil** — and every stage gets caught. Red glow when the technique is observed; blue glow when the matching detection rule fires. The terminal at the bottom is the same `tail -f /var/log/wazuh/alerts.json | jq -r .rule.description` shape that runs in the lab.

<img src="assets/killchain.svg" alt="Animated kill-chain detection movie — six stages, adversary techniques on top, matching SIGMA rules on the bottom · 30s SMIL loop" />

> hand-coded SVG · SMIL only · no JS · same MITRE technique IDs the lab actually fires on (T1595, T1566.001, T1059.001, T1548.002, T1003.001, T1048.003) · MTTD median ≤ 1.7 s in this run, MTTR median ≤ 0.6 s, 6/6 stages contained.

### `pipeline.mmd`

How a single suspicious sign-in travels from the host to a triaged ticket — the actual flow that runs inside [`home-lab-siem`](https://github.com/y-zahidi/home-lab-siem).

<!-- ctf{2/3} flag{yz_alerts_are_just_questions_we_finally_answered} — nice. one to go: open the portfolio, hit `:` and try `:flag`. -->

```mermaid
flowchart LR
  classDef src   fill:#0a1424,stroke:#1f2f4a,color:#79e2ff;
  classDef proc  fill:#08111d,stroke:#5cf2c1,color:#5cf2c1;
  classDef ti    fill:#08111d,stroke:#febc2e,color:#febc2e;
  classDef sink  fill:#08111d,stroke:#79e2ff,color:#cfd6e4;
  classDef alert fill:#1a0a0c,stroke:#ff5f57,color:#ff8a8a;

  W[Windows / Sysmon]:::src --> AGT[Wazuh agent]:::src
  L[Linux / auditd]:::src   --> AGT
  FG[FortiGate firewall]:::src --> SYSLOG[syslog-ng]:::src
  NIDS[Suricata · eth1]:::src --> EVE[eve.json]:::src

  AGT --> MGR[Wazuh manager · custom decoders]:::proc
  SYSLOG --> MGR
  EVE --> MGR

  MGR --> CORR{rule + MITRE map}:::proc
  MISP[MISP · CIRCL · abuse.ch]:::ti --> CORR
  VT[VirusTotal lookups]:::ti --> CORR

  CORR -->|low|  IDX[(elastic index · 30d hot)]:::sink
  CORR -->|med| TRI[triage queue]:::sink
  CORR -->|high|PAGE[on-call page]:::alert

  IDX --> DASH[Kibana / portfolio dashboard]:::sink
  TRI --> DASH
  PAGE --> DASH
```

---

## 02 · detections

### `sample.{xml,yml,rules}` — three syntaxes, one threat

Same impossible-travel sign-in pattern, written for **Wazuh**, **Sigma**, and **Suricata**, because real detection means picking the right layer for each signal. All three map to MITRE [`T1078`](https://attack.mitre.org/techniques/T1078/) (*Valid Accounts*).

<details open>
<summary><b>wazuh</b> · <code>rules/100210-imp-travel.xml</code> · authentication layer</summary>

```xml
<rule id="100210" level="10">
  <if_group>authentication_failures</if_group>
  <same_source_ip />
  <same_user />
  <different_geoip />
  <description>Impossible-travel sign-in: same user, two countries &lt; 1h</description>
  <mitre>
    <id>T1078</id>
    <tactic>Initial Access</tactic>
  </mitre>
</rule>
```

</details>

<details>
<summary><b>sigma</b> · <code>rules/imp_travel_signin.yml</code> · cross-SIEM portable</summary>

```yaml
title: Impossible Travel Sign-in
id: 7e4a2c1e-9a0b-4b1d-8f3a-7c1f0a4d9b21
status: stable
description: Same user, two distinct GeoIP countries within 60 minutes.
author: Yassir Zahidi
references:
  - https://attack.mitre.org/techniques/T1078/
logsource:
  product: azure
  service: signinlogs
detection:
  selection:
    eventName: 'UserLoggedIn'
  timeframe: 60m
  condition: selection | count(distinct GeoIPCountry) by UserPrincipalName > 1
falsepositives:
  - VPN egress switch
  - Travelling executives (allowlist via UserPrincipalName)
level: high
tags:
  - attack.initial_access
  - attack.t1078
```

</details>

<details>
<summary><b>suricata</b> · <code>rules/local.rules</code> · network layer (Tor C2 fallback)</summary>

```text
alert tls $HOME_NET any -> $EXTERNAL_NET any ( \
  msg:"YZ TOR exit-node TLS handshake from internal host"; \
  flow:established,to_server; \
  tls.cert_subject; content:"CN="; nocase; \
  pcre:"/CN=([A-Za-z0-9]{16,})\.onion/i"; \
  threshold:type both, track by_src, count 1, seconds 600; \
  classtype:policy-violation; sid:9000111; rev:1; \
  metadata:mitre_attack T1090.003; )
```

</details>

[See the live transpiler on the portfolio →](https://y-zahidi.github.io/#detect) — same sigma rule compiled live to Wazuh / Splunk SPL / KQL / Suricata as you type.

### coverage matrix

The grid is honest: **bright = a rule is shipping today**, **dim = on the roadmap**, **black = out of scope for the home lab** (e.g. air-gap exfil, hardware initial-access). New cells light up as i push to `home-lab-siem`.

<img src="assets/coverage.svg" alt="MITRE ATT&CK detection coverage matrix — 12 tactics, current shipping coverage shaded by intensity" />

### incident-response playbook

What happens between *"a rule fired"* and *"the ticket is closed"*. Six stages, with the time budget i hold myself to before escalating. Same shape whether it's an SSH brute-force or an LSASS dump.

<img src="assets/playbook.svg" alt="Incident response playbook — alert → triage → contain → eradicate → recover → review with time budgets per stage" />

---

## 03 · evidence

### one year of detection engineering

Each cell is a day; darker cells = higher-volume weeks. The blue cursor sweeps left-to-right on a 20-second loop because the backlog never sleeps. *Numbers come from a deterministic seed so the page is stable; the **shape** (winter peak, weekend dip, ship-streak in the back half) is how the lab actually runs.*

<img src="assets/heatmap.svg" alt="Detection-engineering activity calendar — 52 weeks × 7 days · 687 contributions · 28 in best week · 12 median weekly" />

### what a shift looks like — `tail -f /soc/console`

The KPI tiles on the left are the metrics i'd check first thing on a real shift: alerts/24 h, EPS, MTTD, IOC feed health, lab uptime. The streaming log on the right is the shape the production rules emit; only the timestamps move.

<img src="assets/console.svg" alt="Live SOC console mock — KPI tiles, streaming alert tail, top-source bar chart" />

### lab uptime — `/uptime`

Seven days of synthetic uptime per component, last 50 alerts feed.

<img src="assets/uptime.svg" alt="Lab uptime panel — wazuh-mgr, suricata, fortigate, sysmon, MISP, VT, CIRCL with 7-day sparklines" />

---

## 04 · history

<details open>
<summary><b>Préfecture de Tétouan — Ministère de l'Intérieur (SSIC)</b> · <i>Cybersecurity Intern · 02 May → 31 May 2024</i></summary>

> *Real ministry network. Real perimeter. Real consequences.*

- Helped design and deploy a **multi-layer SIEM** on the production segment: Wazuh manager + Suricata IDS + Sysmon (Windows agents) + MISP threat-intel + VirusTotal lookups.
- Wrote **custom Wazuh decoders** for FortiGate syslog so we'd stop losing fields on rotation; mapped Suricata EVE alerts to MITRE ATT&CK in the alert pipeline.
- Wired MISP **CIRCL + abuse.ch** feeds on a 6-hour sync, plumbed VirusTotal hashing for any Sysmon `EventID=1` with a non-signed binary parent.
- Weekly **Nessus** scans against the perimeter, triage report to the SSIC chief.
- The architecture, repackaged so anyone can `docker compose up`, lives at [`home-lab-siem`](https://github.com/y-zahidi/home-lab-siem).

</details>

<details>
<summary><b>ALTEN Maroc — Tétouan Shore</b> · <i>IT Support Technician (N1/N2) · Mar 2025 → Sep 2025</i></summary>

- **~70 Windows + VPN users** supported, 50+ tickets resolved end-to-end.
- Workstation hardening (BitLocker rollout, GPO baselines), incident response on three credential-phishing cases.
- Wrote runbooks the next intern still uses.

</details>

### projects

| project | stack | what it actually does |
|---|---|---|
| **[home-lab-siem](https://github.com/y-zahidi/home-lab-siem)** | Wazuh · Suricata · Sysmon · MISP · Docker | the internship architecture, packaged. `docker compose up` and the SOC is live. |
| **[ctf-writeups](https://github.com/y-zahidi/ctf-writeups)** | Markdown | TryHackMe / HTB walkthroughs — methodology over flags, one consistent template. |
| **[pentest-cheatsheet](https://github.com/y-zahidi/pentest-cheatsheet)** | Markdown | the cheatsheet i actually use — recon → AD → web → post-ex. |
| **[water-stress-morocco-analytics](https://github.com/y-zahidi/water-stress-morocco-analytics)** | MySQL · QlikView · star schema | DWH on water stress in Morocco — 68k rows, 12 regions, 2015–2025. |
| **[FacturationPro-Enterprise](https://github.com/y-zahidi/FacturationPro-Enterprise)** | C++ · VCL · MySQL | Windows desktop billing — multi-user, role-based, PDF export. |

Also: [HTMLCamp](https://github.com/y-zahidi/HTMLCamp) · [Rabat-Cultural-Website](https://github.com/y-zahidi/Rabat-Cultural-Website)

---

## 05 · toolchain

<img src="assets/stack.svg" alt="Stack heatmap — fluency by domain (detection, threat-intel, perimeter, offensive, OS/identity, languages, infra)" />

### certifications

| issuer | credential |
|---|---|
| Cisco | CCNA 1, 2 & 3 |
| Fortinet | FCF · NSE 1 · NSE 2 · NSE 3 |
| EC-Council | DFE *(Digital Forensics)* · EHE *(Ethical Hacking)* · NDE *(Network Defense)* |
| ICSI | Certified Network Security Specialist (CNSS) |
| Orange Digital Center | Cybersecurity — Morocco |
| French Embassy | DELF B2 — Diplôme d'Études en Langue Française (2025) |

### principles

```text
1. detection without telemetry is theater
2. the best detection is a friendly red team — attack your own rules first
3. every alert ships with triage steps, a mitre id, and the payload that proves it fires
4. write the runbook before you need it; rehearse it with atomic-red-team
5. an alert nobody reads is worse than no alert
6. document the false-positive that almost fooled you, by name
7. blue is what i train for, red is the gym, purple is the loop — no skipped leg-day
```

---

## 06 · reach

Four channels. First reply within 24 h, async-first, time zone Africa/Casablanca.

<img src="assets/contact.svg" alt="Secure contact channel — portfolio, mail, linkedin, github, with response SLA and a signed fingerprint" />

<table>
  <tr><td><b>web</b></td><td><a href="https://y-zahidi.github.io"><code>y-zahidi.github.io</code></a></td><td>portfolio · live SOC console mock · sigma playground</td></tr>
  <tr><td><b>mail</b></td><td><a href="mailto:yassirzahidi8@gmail.com"><code>yassirzahidi8@gmail.com</code></a></td><td>roles · pentest gigs · interesting CTFs</td></tr>
  <tr><td><b>net</b></td><td><a href="https://www.linkedin.com/in/yassir-zahidi/"><code>linkedin/in/yassir-zahidi</code></a></td><td>résumé · recommendations · timeline</td></tr>
  <tr><td><b>code</b></td><td><a href="https://github.com/y-zahidi"><code>github.com/y-zahidi</code></a></td><td>everything you can read on this page</td></tr>
  <tr><td><b>cv</b></td><td><a href="https://y-zahidi.github.io/resume.json"><code>resume.json</code></a></td><td>JSON-Resume schema · machine-readable</td></tr>
</table>

```text
$ exit 0
connection closed.  thanks for reading the receipts.
```

<!--
  ───────────────────────────────────────────────────────────────────────
   if you read source you're already halfway there.
   ctf{1/3} → flag{yz_read_my_decoders_-_let_us_talk}
   hint    → ctf{2/3} hides in the pipeline.mmd block, above
   hint    → ctf{3/3} hides on the portfolio site, behind a vim cmdline
   signal me. the coffee is on me.
  ───────────────────────────────────────────────────────────────────────
-->
