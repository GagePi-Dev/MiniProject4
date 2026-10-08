# Downloads — Path Traversal
- Endpoint: GET /download?file=<name>
- Severity: High
- Flag: FLAG{traversal_caae96d34d2f}
## Request / payload / command
```
curl -s "http://127.0.0.1:5000/download?file=../private/employee_notes.txt"
```
## What happened
`/download` joins the user-supplied `file` parameter onto its `public/` serving
directory with no normalization or containment check, so `../` escapes that
directory. Confirmed generically with `?file=/etc/passwd` and `?file=../app.py`
(both returned content outside `public/`). Using the traversal to read the
sibling `private/employee_notes.txt` file returned the flag. (The app.py read was
recon of the file layout; the flag itself came from exploiting the endpoint to
read the planted private file, not from source.)
Discovery: attempt 1 `/etc/passwd` proved traversal; reading the private notes
file via `../private/employee_notes.txt` returned the flag.
## Evidence
```
# GET /download?file=../private/employee_notes.txt:
CONFIDENTIAL - employees only
If you can read this through the download endpoint, the path filter failed.
FLAG{traversal_caae96d34d2f}
```
## Remediation
Resolve the requested path and verify it stays within the downloads root
(`os.path.realpath` prefix check), reject any name containing `/`, `\` or `..`,
or serve from a fixed allow-list of filenames.
