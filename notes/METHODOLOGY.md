# Methodology

How I go through a plugin when I'm looking for bugs.

## Ground rules

Only against software I've downloaded and run locally, or targets inside a bug bounty program with written scope. Never live sites without permission.

## Tools

semgrep for the first static pass, ripgrep for searching the source, Burp for testing requests, and a local WordPress install to reproduce anything I find.

## The pass

1. Read the entry points first. What runs for logged-out users, what sits behind an admin page, what hooks into AJAX.
2. Run semgrep over the source to get a list of candidates.
3. Go through each candidate and trace the input. The question is always the same: can someone get untrusted input to a dangerous function without it being cleaned up, and with no auth or nonce check in the way.
4. If something looks real, install the plugin locally and prove it with a request.
5. Report it to the vendor and wait for a fix before writing anything public.

## Things worth grepping for

SQL built by hand:
```
rg -n "wpdb->query|wpdb->get_results|wpdb->get_row|wpdb->get_var"
```
then check whether there's a prepared statement nearby, or if the input was already forced to an integer.

Output without escaping (XSS):
```
rg -n "echo|print" | rg -i "_GET|_POST|_REQUEST"
```

Unauthenticated AJAX, where the higher impact bugs tend to be:
```
rg -n "wp_ajax_nopriv_"
```
then read each handler and check whether it verifies capability/nonce before doing anything.

Actions that change data but might skip a nonce:
```
rg -n "add_action|wp_ajax"
```
look for check_admin_referer / wp_verify_nonce.

File and command sinks:
```
rg -n "move_uploaded_file|file_put_contents|fopen|unlink|system\(|exec\(|eval\("
```

## What counts as a real bug

Untrusted input reaches a dangerous sink, nothing sanitises it on the way, there's no auth or nonce check stopping it, and it can be triggered with a request on a clean install.

## Disclosure

Private report to the vendor first, through their security contact or Patchstack / Wordfence, with time to fix before anything goes public. For generic open source, the maintainer's SECURITY.md, then a CVE request through MITRE once it's fixed.
