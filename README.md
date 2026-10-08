# WordPress Plugin Security Research

Write-ups from my work looking for vulnerabilities in open source WordPress plugins.

Everything is done against plugins I download and run locally. I don't test live sites. If I find a real bug I report it to the vendor first and wait for a fix before writing anything public about it.

## How I approach it

Static scanners flag a lot of things, and most of them are nothing. The actual work is taking each flagged spot and following the user input to see if it can reach something dangerous without being cleaned up first. Usually it can't, and the interesting part is working out why.

For each plugin I look at a few things:

- Endpoints that run for logged-out users (unauthenticated AJAX). These are where the high impact bugs live.
- SQL queries built by hand instead of with prepared statements.
- File upload and file write code, and whether the type or path can be abused.
- Output that includes user input without escaping it (XSS).
- Actions that change data but skip the nonce or capability check (CSRF / access control).

Then I trace each one properly. A query without a prepared statement is worth a look, but it's not automatically a bug. You have to see what the input goes through first.

## Confirmed advisories

Any confirmed and fixed issues get a full write-up in [`advisories/`](advisories/), one folder per CVE. None published yet.

## Disclosure

If something real turns up here, it goes to the vendor through their security contact or a coordinating body (Patchstack / Wordfence), with time to fix before anything is published.
