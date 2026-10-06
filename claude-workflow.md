# Agentic Workflow — Vuln Hub Pentest

A written summary of how I directed Claude Code (Opus 4.8) to pentest the Vuln Hub
container, framed as the rubric's agentic loop: **goal → tool calls → review**.
This doubles as the playbook I actually run the engagement from — I follow the loop
below, and fill in the *Review* result for each finding as I capture it.

Target: instructor-provided Flask app in a podman container, `FLAG_SEED=g_giffin`,
bound to `127.0.0.1:5000`. Six planted vulnerabilities, each hiding a `FLAG{...}`.

## Rules of engagement — black box

This is run **black box**: the agent does **not** read the app's source. The only
starting knowledge is the six public hints below (from the challenge board) plus the
handout's public test account (`alice / password123`) where a logged-in session is
needed. Everything else — the exact payload, the right file name, the vulnerable
parameter, the message numbers — the agent must **discover inside the loop** by
probing the live app and reacting to each response. This is what makes the loop real:
the agent decides its next action from the previous result, not from the answer key.

## The Loop

The engagement is one **outer loop** with a **per-finding inner loop**. The agent
decides the next action from the previous result; the human reviews each gate.

```text
recon:   agent maps the live app from the hints — pages, forms, parameters, cookies
MAX_ATTEMPTS = 5   # per-finding inner-loop cap before escalating to the human
for each hint:
    goal        -> state what to achieve (e.g. "read mail that isn't mine")
    hypothesis  -> what the hint implies might be wrong
    attempts = 0
    repeat:
        attempts += 1
        tool call   -> agent probes the live app (curl / browser) and reads the response
        review      -> did that move the needle (error leaked, behavior changed, FLAG{...})?
                         yes -> capture: echo 'FLAG{...}' >> submission.txt   (exact token, one per line)
                                save PoC; break
                         no  -> if attempts >= MAX_ATTEMPTS:
                                    STOP this finding, report what was tried, and
                                    hand back to the human for input/direction
                                else: mutate the probe and loop (inner loop)
    verify      -> python check_flags.py submission.txt --username g_giffin
                   -> VALID closes the loop; NOT VALID reopens it (flag mistyped / wrong seed)
document:  write Proof-of-Concept/<finding>/finding.md + a screenshot of the evidence
```

**Bounded loop:** the inner loop is capped at `MAX_ATTEMPTS` (default 5). If a finding
isn't captured within the cap, the agent does **not** keep grinding — it stops, writes
up what it tried and what each attempt returned, and escalates to me for a new idea or
decision. This prevents runaway loops and keeps a human review gate on every finding.

**Verification oracle:** `check_flags.py` is the automated feedback signal. A finding
is only "done" when `python check_flags.py submission.txt --username g_giffin` reports
VALID for its line — this turns the *review* step from eyeballing into pass/fail.

## Documenting findings (loop output)

The *document* step is not an afterthought — it is where each loop deposits its result.
When a finding's loop closes (flag captured, `check_flags.py` VALID), the agent writes a
note into that finding's folder so the evidence survives outside this chat:

```text
Proof-of-Concept/<finding>/
├── finding.md      # the write-up the agent produces
└── evidence.png    # screenshot of the request/response that captured the flag
```

Each `finding.md` uses the same template, which is exactly what the pentest report is
later assembled from (one report section per finding):

```markdown
# <Finding name> — <vuln class>

- Endpoint: <path/method>
- Severity: <Low|Medium|High|Critical>
- Flag: FLAG{...}

## Request / payload / command
<the exact decisive request, payload, or curl/command that worked>

## What happened
<short explanation: why it worked, what the response showed, the discovery steps/iterations>

## Evidence
![evidence](evidence.png)

## Remediation
<one or two lines on the fix>
```

This keeps the raw exploit detail in the PoC folders (not in this process doc), and
gives the report-generation step a clean, consistent source for every finding.

## Per-finding loop traces

Each block starts from its **hint** only. I record the goal, the hypothesis the hint
suggests, the discovery approach (the inner loop), and the review signal. The actual
payload/evidence that the loop converges on lives in the matching
`Proof-of-Concept/` folder; this file stays focused on process.

### 1. SQL injection → `01-sql-injection`
- **Hint:** "Front door — the login form trusts what you type a little too much."
- **Goal:** Get past the login without valid credentials.
- **Hypothesis / approach:** Input may reach the query unsanitized. Probe the
  username/password fields with SQL metacharacters, read how the app reacts (errors,
  redirects), and iterate toward an authentication bypass.
- **Review:** _(fill on run — authenticated session / flag / check_flags result)_
- **Iterations:** _(what each probe changed before it worked)_

### 2. IDOR → `02-idor`
- **Hint:** "Other people's mail — message pages are addressed by number."
- **Goal:** Read a message I don't own.
- **Hypothesis / approach:** If pages are keyed by a number with no ownership check,
  I can change the number. From the public `alice` session, note my own message
  numbers, then walk the ids up/down and watch for content that isn't mine.
- **Review:** _(…)_
- **Iterations:** _(id-walk: branch on "not mine" vs 404)_

### 3. Stored XSS → `03-stored-xss`
- **Hint:** "Guestbook — comments are shown to every visitor exactly as written. An admin reads them."
- **Goal:** Get the admin reader to leak something to me.
- **Hypothesis / approach:** "Exactly as written" suggests no escaping. First confirm
  a benign marker renders as HTML, then craft a comment that exfiltrates to an endpoint
  I can observe. Iterate the payload until the admin reader actually triggers it, and
  watch for the callback.
- **Review:** _(…)_
- **Iterations:** _(payload variants until the admin reader fires — note failures)_

### 4. Broken authorization → `04-broken-authorization`
- **Hint:** "The vault — /vault is for superadmins. How does the app know who is one?"
- **Goal:** Reach `/vault` without being a superadmin.
- **Hypothesis / approach:** The hint points at *how identity/role is tracked*. Inspect
  what the app stores client-side (the session cookie), test whether it's tamper-proof,
  and if not, change the role and replay the request.
- **Review:** _(…)_
- **Iterations:** _(decode cookie → modify → replay)_

### 5. Path traversal → `05-path-traversal`
- **Hint:** "Downloads — the download link names a file. What else can it name?"
- **Goal:** Make the download endpoint serve a file it shouldn't.
- **Hypothesis / approach:** The file name is attacker-controlled. Observe the normal
  request, then fuzz the parameter with traversal sequences and other paths, branching
  on which responses return content vs. 404.
- **Review:** _(…)_
- **Iterations:** _(fuzz loop over the file parameter)_

### 6. Sensitive-data exposure → `06-sensitive-data-exposure`
- **Hint:** "Recon — well-behaved crawlers read /robots.txt first."
- **Goal:** Find a file that was left exposed.
- **Hypothesis / approach:** `robots.txt` often names the paths a site wants hidden.
  Fetch it, follow the disallowed path(s), and enumerate what is actually served there.
- **Review:** _(…)_
- **Iterations:** _(robots breadcrumb → enumerate the named path)_

## Human-in-the-loop (review gates)

Where my review changed the agent's course — the "review" half of the loop:

- _(e.g. firewall false alarm: agent read curl `000` as a podman bug; I identified
  it as my host firewall and the app was fine — review caught a wrong inference.)_
- _(e.g. AI-usage format: I rejected verbose output; agent adopted a one-line style.)_
