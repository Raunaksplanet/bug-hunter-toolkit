# P1 Hunting Methodology

> Lessons extracted from 7 verified P1 reports across AbaClient, CoinPoker, and other targets.
> These patterns repeat. Learn them, and you'll see them everywhere.

---

## Core Truths

- **No single bug is P1.** Every P1 is a chain of 2-4 bugs that individually are P4-P2.
- **You're looking for the missing check.** Not what the code does — what it *doesn't* do.
- **Read every auth/validation function twice.** The bug is usually in what happens *after* it returns.
- **State machines are gold.** Transitions between states often skip validation. 302 redirect, then call the handler without re-checking trust.

---

## The 5 Recurring Patterns

### Pattern 1: Trust That Doesn't Travel

The most common P1 pattern. A trust decision is made for URL A, then a redirect/forward/IPC message sends the user to URL B **without re-validating trust**.

**Where to look:**
- `validateFirstTimeHTTP()` — called once, then never again after redirect
- `shouldMakeNewConnection:` — returns YES, then never checks again
- `webSecurity: true` — but bypassed via IPC channel that routes to `file://`
- Whitelist hosts — check is done at entry, redirect target is not checked

**Report examples:**
- AbaClient Mode B: `validateFirstTimeHTTP()` called for HTTPS, then 302 → HTTP bypasses it entirely
- AbaClient Mode A: whitelist check at entry, but HTTP redirect to any host is trusted
- CoinPoker: `webSecurity` blocks `file://` from web origin, but IPC `send('EVT_OPEN_URL')` bypasses it to the lobby's `file://` origin

### Pattern 2: Validation That Is Just For Show

Validation that looks correct but can be trivially bypassed.

**Variations:**
- **Magic byte check only** — JPEG validator: "starts with FF D8 FF, ends with FF D9, >= 10KB". Any file with those bytes passes.
- **Unsigned JSON** — Update manifest parsed with no signature. If you can MITM the DNS, you control the update.
- **Env var bypass** — `DISABLE_VALID_ABACUS_CODE=1` skips all JAR signature verification.
- **Folder existence = trust** — `if (folder.exists() && folder has files) → trust`. Created by first connection, never invalidated.
- **Type check but not protocol** — `isString` check but no content validation. MessageType=0 crashes the server.

**Question to ask:** "Can I pass this check without actually having what it's supposed to verify?"

### Pattern 3: Where Input Becomes Command

Look for every place where user-controlled input crosses into:
- Shell execution
- File path construction
- IPC message
- eval/new Function
- Dynamic class loading

**The chain:** JSON field → part of command string → no escaping → shell injection.

**Common gaps:**
- macOS bash wrappers (`ProcessBuilder` on Linux, `/bin/sh` script on macOS)
- Path construction for file writes (`download-image` IPC → filename from URL → `../` traversal)
- JVM arguments (`-D` flags from JSON)
- Classpath ordering (attacker's JAR first shadows signed JAR)

**Report examples:**
- AbaClient: `applicationArguments` from JSON → `StringUtil.concatWithDelimiter(" ", cmd)` → `/bin/sh` → RCE
- CoinPoker: URL filename → `image-cache/{basename(url.pathname)}` → arbitrary file write

### Pattern 4: IPC Without Borders

IPC channels that cross privilege boundaries without proper gating.

**What to look for:**
- `send()` without whitelist vs `invoke()` with whitelist
- Any IPC handler that accepts a URL/path and opens/navigates to it
- IPC messages that trigger file operations with attacker-controlled filenames
- Mix of `file://` and `http://` origin handling in the same app

**The bug:** Preload exposes `send(channel, data)` with no channel whitelist → renderer sends to any registered handler → main process executes with full privileges.

**Report examples:**
- CoinPoker: `preload.send()` has no whitelist → `EVT_OPEN_URL` → `file://` origin bypass

### Pattern 5: Deserialization Without Boundaries

JSON/XML/plist deserialization where the types are trusted implicitly.

**What to look for:**
- Gson/JAXB/Jackson without `@JsonFilter` or `@JsonIgnoreProperties`
- Plist deserialization without schema validation
- Type adapter that calls `getAsString()` without checking `isJsonPrimitive()`
- Fields that are arrays/lists expected to be one type but attacker provides another

**The bug:** JSON `{"field": 0}` where code expects a string → crash or type confusion.

**Report examples:**
- AbaClient: Gson deserialization of launch config — no validation on any field
- AbaClient: `STRJarInfoContainer.equals()` NPE when `md5Hash` is null in JSON
- Sophos IPC: `{'MessageType': 0}` → crash (expected string, got int)

---

## Hunting Flow

### Step 1: Map the Trust Boundaries

1. List every IPC channel, URL handler, socket, and service endpoint
2. For each: where is trust established? Where is it NOT re-validated?
3. Mark every state transition — after a redirect, after an IPC send, after a file load

### Step 2: Find Every Validation Gate

1. `AuthorizationExternalForm` checks
2. `validateFirstTimeHTTP()` / trust dialogs
3. Code signing validation (`isSophosSignTypeForConnection:`)
4. Whitelist/blacklist checks
5. Magic byte / header validation

For each: **what happens if you skip past it?** Can you reach the protected code path without passing the check?

### Step 3: Trace Input to Sink

Every JSON field, plist key, URL query param, IPC message payload → trace to:
- File write/read
- Shell execution
- Network request
- Dynamic class loading
- eval/new Function

If there's no sanitization between source and sink, you have a bug.

### Step 4: Chain Everything

- Info disclosure + auth bypass + command injection = P1
- DoS + persistent config + no re-auth = P1
- SSRF + file write + LFI = P1
- IPC abuse + no whitelist + file:// origin = P1

**Rule:** If you have two bugs that share a data flow, try to chain them.

---

## Checklist Per Service

- [ ] Auth function has skip path? (env var, missing call after redirect, null check bypass)
- [ ] JSON deserialization with type confusion potential? (int for string, null for object)
- [ ] macOS bash wrapper for command exec? (JRERunner, NSTask via shell)
- [ ] IPC `send()` without channel whitelist?
- [ ] Trust established once but not re-validated on redirect/state change?
- [ ] File path constructed from user input? (path traversal, null byte)
- [ ] Unsigned update/configuration download?
- [ ] Magic-byte-only file validation? (JPEG polyglot)
- [ ] Classpath/JAR loading with attacker-controlled paths?
- [ ] Protocol downgrade possible? (HTTPS → HTTP follows 302)
- [ ] World-writable socket/port that accepts commands?
- [ ] Plist deserialization without schema?
- [ ] ObjC `respondsToSelector:` abuse? (KVC, forwardInvocation)
- [ ] Entitlement check that can be bypassed? (hardcoded Team ID, not verifying)
