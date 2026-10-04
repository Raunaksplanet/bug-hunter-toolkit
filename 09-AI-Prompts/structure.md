# Bug Bounty Session Structure

> **okay now you can start with reverse engineering all apps and i dont expect bugs in first 4 to 5 deep scans, its very old and highest bounty program, we are in competition with top world hackers, take your time, do deep code review, look for chains, document all p5 p4 p3 p2 p1 vuln, hunt for impact and crits only**

## Directory Layout

```
NN-ProjectName/
├── 01-Recon/          — Initial reconnaissance, service enumeration, process mapping
│                        launchctl, nm,otool, strings, binary analysis pre-re
├── 02-Credentials/    — Auth tokens, creds, API keys found during recon
├── 03-Tools/          — PoC scripts, fuzzers, test programs (Python, ObjC, C)
├── 04-Docs/           — Markdown findings reports, algorithm notes, poc writeups
├── 05-Apps/           — Target application bundles, plists, configs
└── 06-Installer/      — Installer packages, dmgs, extracted payloads
```

## Reference Methodology

> **Read [`hunting-methodology.md`](hunting-methodology.md) before starting — it contains the 5 recurring P1 patterns extracted from 7 verified reports. Use it as your mental checklist during every phase.**

## Workflow

### Phase 1: Recon (01-Recon/)
1. **Enumerate services** — `launchctl print system`, Mach ports, XPC services, Unix sockets
2. **Identify processes** — `ps aux`, process tree, user vs root daemons
3. **Map attack surface** — World-writable sockets, user-accessible Mach ports, XPC brokers
4. **Document architecture** — Process roles, IPC mechanisms, trust boundaries

### Phase 2: Binary Analysis (01-Recon/ per-target)
1. **Static analysis** — `nm`, `otool`, `strings`, Hopper/IDA/Ghidra on each binary
2. **Class dump** — ObjC class hierarchies, protocols, ivars
3. **Auth mechanism** — `SAVConnectionAuthenticator`, `SMECodeSignValidator`, `AuthorizationExternalForm`
4. **IPC protocol** — Message formats, framing, serialization (plist, NSPortMessage, XPC)
5. **Check [`hunting-methodology.md`](hunting-methodology.md) patterns** — trust that doesn't travel, validation that is just for show, input becomes command

### Phase 3: Tooling & PoC (03-Tools/)
- **IPC fuzzer** — malformed plists, type confusion, length overflow, path traversal
- **Protocol reverse** — decode message types, auth tokens, command IDs
- **Auth bypass** — spoof clientProfileData/systemProfileData, brute force, null auth

### Phase 4: Documentation (04-Docs/)
- **NN-finding-title.md** — Per-finding writeup with PoC, reproduction, impact
- **NN-findings-summary.md** — Consolidated list of all findings with severity
- **NN-auth-algorithms.md** — Auth mechanism reversal notes

## Key Checks for Every Service

- [ ] World-writable Unix socket? → fuzz protocol, try auth bypass
- [ ] User-accessible Mach port? → connect, enumerate DO/XPC methods
- [ ] Code signing validation? → look for bypass (self-signed, team ID match)
- [ ] Auth token comparison? → constant-time? predictable? brute-forceable?
- [ ] AuthorizationExternalForm? → can we craft without Security framework?
- [ ] Entitlements? → com.apple.security.* grants?
- [ ] File permissions? → writable configs, plists, databases?
- [ ] Path traversal? → in OriginalLocation, file paths, service names
- [ ] Type confusion? → non-string MessageType, array for dict, int for enum
- [ ] Race condition? → TOCTOU in auth check, file operations
- [ ] **Trust that doesn't travel**? → redirect/state change skips re-validation
- [ ] **Validation that is just for show**? → magic bytes, env var bypass, folder exists
- [ ] **Input becomes command**? → trace every JSON field → shell/file/IPC
- [ ] **IPC without borders**? → send() without whitelist vs invoke() with whitelist
- [ ] **Deserialization without boundaries**? → type confusion from trusting JSON types

## Mindsets

- **Assume nothing is auth-gated** until you prove otherwise
- **Fuzz everything** — one crash = P1 DoS
- **Chain everything** — auth bypass + command injection = RCE
- **Document everything** — P5 today might be P2 in a chain tomorrow
- **Old code = buggy code** — legacy NSConnection, manual plist parsing, C-style buffer ops

## Scoring Reference

| Sev | Example | Bounty |
|-----|---------|--------|
| P5  | Info disclosure (minor), recon data | ~$500 |
| P4  | Low-impact DoS, minor info leak | ~$1-2k |
| P3  | Medium info disclosure, config leak | ~$2-5k |
| P2  | Auth bypass, limited impact RCE | ~$5-20k |
| P1  | Pre-auth RCE, complete compromise | ~$20-80k |
