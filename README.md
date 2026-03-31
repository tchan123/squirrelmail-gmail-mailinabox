# SquirrelMail for Gmail + Mail-in-a-Box

A custom SquirrelMail build preconfigured for Gmail IMAP/SMTP on a [Mail-in-a-Box](https://mailinabox.email/) server.

## What's included

- Gmail IMAP (`imap.gmail.com:993`, TLS) and SMTP (`smtp.gmail.com:587`, STARTTLS) preconfigured
- Folder mappings for Gmail's special folders (Trash, Sent, Drafts)
- Fix for nginx `fastcgi_split_path_info` stripping the `/squirrelmail` prefix, which caused POST→GET redirects and silently broke delete, move, and other actions
- User data stored at `/home/user-data/squirrelmail/` (standard Mail-in-a-Box layout)

## Per-instance configuration

After cloning to `/usr/local/lib/squirrelmail/`, edit `config/config.php` and update:

| Setting | Variable | Example |
|---|---|---|
| Your domain | `$domain` | `example.com` |
| Organization name | `$org_name` | `Webmail` |

Users log in with their **Gmail address** as the username and a **Gmail App Password** (not their Google account password). Generate one at [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords).

## nginx

Serve SquirrelMail at `/squirrelmail/` via PHP-FPM. The `include/init.php` fix handles the `SCRIPT_NAME` prefix issue automatically — no nginx config changes needed.

## Changes from upstream

### `include/init.php`
Fixed a bug where actions like deleting or moving an email appeared to do nothing — the page would refresh but the message was still there.

When a user clicks Delete (or Move, or any button that submits a form), the browser sends a POST request containing the selected messages. On a Mail-in-a-Box server, nginx was silently redirecting that request to a slightly different URL, and browsers responding to a redirect automatically switch from POST to GET — dropping the list of selected messages in the process. SquirrelMail received the request with no messages selected, so nothing was deleted.

The root cause was that nginx strips the `/squirrelmail` prefix from the URL before handing it to PHP, causing SquirrelMail to build incorrect form action URLs. This affects any Mail-in-a-Box installation regardless of mail provider. Since the Mail-in-a-Box nginx config is auto-generated and cannot be edited directly (it gets overwritten on updates), the fix lives in SquirrelMail itself — it detects the missing prefix and corrects it before any page logic runs.

### `config/config.php`
Preconfigured for Gmail:
- IMAP: `imap.gmail.com:993`, TLS enabled, server type `gmail`
- SMTP: `smtp.gmail.com:587`, STARTTLS (`use_smtp_tls = 2`), auth mechanism `login`
- Folder mappings: Trash → `[Gmail]/Trash`, Sent → `[Gmail]/Sent Mail`, Drafts → `[Gmail]/Drafts`
- User data paths set to Mail-in-a-Box standard: `/home/user-data/squirrelmail/`

---

# SqirrelMail 1.5.2

A version of SquirrelMail for PHP 7.0 and above with additional changes for compatibility or security reasons.


This version contains the following changes:
  * Legacy constructors replaced with `__construct`.
  * While/List/Each are now `ForEach`.
  * Instances of `mt_rand` are now cryptographically secure, using `random_int`.
  * Message IDs are generated differently.
    * The ID is now a Version 5 UUID based off 64 cryptographically secure random bytes.
    * The domain now matches the value of the username variable, if it contains an "@". Otherwise, it falls back to the SERVER_NAME variable like normal.
  * Instances of `SizeOf` are now `StrLen` because PHP is not C.
  * Instances of `create_function` are now inline functions.
  * The `X-Frame-Options: SAMEORIGIN` header has been replaced with CSP's `frame-ancestors` header. This is set based on the `provider_uri` preference, and accepts any http or https domains or subdomains that match.
    * A new variable in the config will be added at a later point.
  * SCRAM support for SMTP and IMAP logins, supporting any hash algorithm PHP supports (with checks against HMAC list on 7.2+).
  * Optional parameters corrected for PHP 8.0.

Additionally, some minor fixes that would cause warnings or strange failures have also been resolved.
Changes to the official source code are tracked in the `trunk` branch and merged with `master` as soon as possible. I'm looking into a way to automate this process.
