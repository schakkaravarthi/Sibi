# PIN security deployment setup

This package adds a shared PIN gate to Sibi Cash Lender.

## What it does
- Requires the normal family PIN before the dashboard can be used.
- Provides a Change PIN action inside the dashboard. The master PIN authorizes changing the normal PIN.
- Stores only a salted normal-PIN hash in Supabase.
- Keeps the master PIN and session-signing secret in Netlify environment variables, not in the browser bundle or GitHub source.
- Protects ledger API reads and writes with a signed session token.
- Locks access after repeated incorrect attempts. A PIN change increments a version number and invalidates existing sessions.
- Does not create family accounts or modify existing ledger rows.

## Before deploying
Do these steps before switching the Netlify production deploy to this branch.

### 1. Run the Supabase SQL
Open the Supabase SQL Editor for the existing project. Run the contents of:
`CashLender-Supabase-Netlify/supabase-setup.sql`

The script adds the `public.cash_lender_security` table and its access policies. It does not delete or rewrite existing ledger records.

### 2. Add Netlify environment variables
In Netlify, open the `sibiaccount` site's environment variables and add the following variables for the Production deploy context. Ensure their scopes include Functions:

- `CASHLENDER_INITIAL_PIN`: the normal PIN supplied by the site owner. Used only to initialize the security row on first use if it does not exist.
- `CASHLENDER_MASTER_PIN`: the master PIN supplied by the site owner. Keep this only in Netlify.
- `CASHLENDER_SESSION_SECRET`: a new random secret of at least 32 bytes. Generate one in a terminal using `openssl rand -hex 32`.

Keep the existing `SUPABASE_URL` and `SUPABASE_SERVICE_ROLE_KEY` variables. Do not put any secret or PIN in GitHub, HTML, JavaScript delivered to the browser, or a public issue.

After the first successful login initializes the normal-PIN hash in Supabase, you may remove `CASHLENDER_INITIAL_PIN` from Netlify to prevent it being used as a bootstrap value again. Do not remove `CASHLENDER_MASTER_PIN` or `CASHLENDER_SESSION_SECRET`.

### 3. Deploy
After the SQL and environment variables are in place, trigger a fresh Netlify production deploy using this branch, `pin-security-v5`, or merge this branch into the configured production branch. Environment-variable changes require a new deploy to be picked up by Functions.

### 4. Verify
- Open the website in a private/incognito window and confirm the PIN screen appears.
- Verify the normal PIN unlocks the dashboard.
- Use Change PIN and enter the master PIN to set a new normal PIN.
- Confirm the previous normal PIN no longer works and that the new one does.
- Confirm the same login works on another family device.

## Session and local-data notes
The browser remembers an authenticated session for up to one year unless the user locks manually, the PIN changes, or browser site data is cleared. The dashboard's local browser cache is not encrypted by this PIN layer; protect access to the underlying device/browser profile as well.

If the SQL or Netlify secrets are missing, login/API access should fail closed. Do not delete or clear the existing ledger table when troubleshooting.
