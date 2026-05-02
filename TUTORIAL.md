# Build a personal web app with Claude Design + Claude Code

A practical tutorial for designing and shipping a small private web app — checklist, tracker, planner, dashboard, whatever — that lives at a real URL and (optionally) syncs across your devices and the people you share it with.

You don't need to know how to code. You will be doing a lot of *copy, paste, click*.

**What you'll have at the end:**

- A custom-designed single-page web app, hosted free at a URL like `your-app.netlify.app`
- (Optional) Cloud-synced state so your phone and laptop stay in lockstep
- (Optional) A shared passcode so 2–5 trusted people can use the same app

**Time:** ~1–2 hours for the basic version. Add 1–2 hours if you do the optional sync + auth phases.

---

## Contents

1. [Tools you'll need](#tools-youll-need)
2. [The pattern, in one paragraph](#the-pattern-in-one-paragraph)
3. [Phase 1 — Design in Claude Design](#phase-1--design-in-claude-design)
4. [Phase 2 — Hand off to Claude Code](#phase-2--hand-off-to-claude-code)
5. [Phase 3 — Deploy to Netlify](#phase-3--deploy-to-netlify)
6. [Phase 4 — (Optional) Cross-device sync via Supabase](#phase-4--optional-cross-device-sync-via-supabase)
7. [Phase 5 — (Optional) Authentication: shared passcode](#phase-5--optional-authentication-shared-passcode)
8. [Phase 6 — Verify and roll out](#phase-6--verify-and-roll-out)
9. [Common pitfalls — one-page reference](#common-pitfalls--one-page-reference)
10. [Prompts cheat sheet](#prompts-cheat-sheet)

---

## Tools you'll need

| Tool | Purpose | Cost |
|---|---|---|
| Claude account | Access to Claude Design and Claude Code | Pro plan ($20/mo) recommended |
| Claude Code | Implements the app, deploys, debugs | Included with Pro |
| Netlify account | Hosts your app at a real URL | Free |
| Supabase account *(optional)* | Cloud database + sync | Free tier sufficient |
| GitHub account *(optional)* | Source control + Netlify auto-deploy | Free |

You also need a modern browser and the willingness to read error messages carefully. That's it.

---

## The pattern, in one paragraph

You **design** the app in Claude Design — a tool that lets Claude make working HTML/CSS/JS prototypes you can interact with. You iterate visually until it feels right, then **hand off** the prototype to Claude Code, which produces a clean, deployable single-file implementation. You **deploy** that file to Netlify in 30 seconds. Optionally, you **add a backend** (Supabase) so state syncs between devices, and a **passcode gate** so a small group can use the same app. Each phase is independent — you can stop after Phase 3 and have a working app, or go all the way to Phase 6 and have a private, multi-user, synced web app.

---

## Phase 1 — Design in Claude Design

**Goal:** Produce a working HTML prototype that captures what your app looks like and how it behaves. Don't worry about backend, deployment, or accounts — just the visual + interaction design.

### Open Claude Design

Go to **claude.ai/design** and start a new project. Give it a clear name (this becomes part of the handoff URL later).

### The opening prompt

The first prompt is the most important one in the whole tutorial. It sets the design direction. Include all of these:

- **What** you're building (one sentence)
- **Why** — the actual use case, not a generic description
- **Aesthetic direction** — colours, mood, typography vibe, references
- **Core features** — 3–7 bullet points
- **Persistence** — say "Persist state via `window.storage`" so Claude Design uses the right API
- **Constraints** — locale (e.g., UK English), device (phone-first?), accessibility, anything else

A good template:

> I want to build a `[type of app]` for `[specific personal use case]`.
>
> Aesthetic direction: `[describe the mood — e.g. "deep plum-black canvas, gold accents, restrained motion, serif typefaces, like an antique almanac"]`. Reference: `[any inspiration you can name]`.
>
> Core features:
> - `[feature 1]`
> - `[feature 2]`
> - `[feature 3]`
> - `[etc.]`
>
> Persist state via `window.storage`. `[Locale, e.g. UK English, no em dashes]`. `[Any other constraint, e.g. "tabular numerals on counters"]`.
>
> Start with a single-column immersive design and we'll iterate from there.

### The audit loop

After Claude Design produces a first version, **don't immediately ask for changes you can think of**. Instead, ask Claude Design to critique its own work:

> Can we audit this as a QC to see if there's anything we need to edit or update or make more user-friendly within the UI/UX?

This is high-leverage. Claude Design will list real issues — things you'd miss as the designer. Each round of audit-and-fix typically improves the design more than the same time spent on free-form requests.

After the audit, reply with which fixes to apply:

> Yes, please apply fixes 1, 2, 4, 6, 7. Skip 3, 5, 8 for now.

Or just: *"Apply all of them."*

### When to declare design done

Stop when the audit comes back with mostly **polish** items rather than **correctness** items. Sample polish items: "the chevron is small," "consider a print stylesheet." Sample correctness items: "two cards render the same data twice," "the today marker is in the wrong position."

Polish items often resolve themselves once you're using the app. Don't iterate forever — production tools will reveal issues that design can't.

---

## Phase 2 — Hand off to Claude Code

**Goal:** Get a clean, deployable single-file HTML implementation of the design.

### Trigger the handoff in Claude Design

In Claude Design, look for an export / handoff option. It produces a URL pointing to a bundle containing the prototype, a README, and the chat history.

### The Claude Code prompt

Open **Claude Code** in any directory you'd like the file to live in. Paste:

> Fetch this design file, read its readme, and implement the relevant aspects of the design. `[paste the handoff URL here]`
>
> Implement: `[main filename from the bundle, e.g. "My App.html"]`

Claude Code will:
1. Fetch the bundle (it arrives as a gzipped tarball; Claude Code unpacks it automatically).
2. Read the README, which tells it to read the chat transcripts and the main file.
3. Produce a single self-contained HTML implementation in your directory.

### What you should explicitly ask Claude Code to strip

The Claude Design prototype includes a few things that are useful during design but not in production:

- **The "Tweaks Panel"** — a live colour/density/layout panel, used for design iteration
- **`window.storage`** — a Claude Design-specific API that should become standard `localStorage`
- **`__edit_mode_set_keys`** postMessage hooks
- **Any `EDITMODE-BEGIN` / `EDITMODE-END` markers**

If Claude Code doesn't strip these on its own, ask:

> Strip the design-tool affordances from the file — TweaksPanel, the postMessage edit-mode hooks, EDITMODE markers, and the Claude Design-specific `window.storage` calls. Replace `window.storage` with `localStorage`. Bake in the chosen tweak defaults as constants.

### Verify the file actually works

Before deploying, ask Claude Code to verify:

> Verify this renders and behaves correctly in a browser using Playwright. Test the main interactions: `[list 2–3 things, e.g. "ticking a checkbox, expanding a card, persistence after reload"]`.

This catches half of all bugs before users see them.

---

## Phase 3 — Deploy to Netlify

**Goal:** Get your file at a real URL anyone can visit.

There are two reasonable paths. Pick one.

### Path A: Netlify Drop (no account, 30 seconds)

Best for: trying it out, one-shot deploys.

1. Go to **app.netlify.com/drop**
2. Drag your `[your-app].html` file onto the drop zone
3. Wait ~10 seconds. You'll get a URL like `random-name.netlify.app`

To rename it from random to something nicer: click the site → **Site configuration** → **Change site name**.

**Updating later:** drag a newer version of the file onto **Deploys** tab → bottom of page (it's smaller than you'd expect — scroll past the deploy list).

### Path B: GitHub + Netlify (better for ongoing work)

Best for: anything you'll update more than twice.

1. Sign up at **netlify.com** using **Sign up with GitHub** (also creates a Netlify account for you)
2. Create a new GitHub repo for your app. Upload your file as **`index.html`** (so the URL is clean)
3. Back in Netlify: **Add new site** → **Import an existing project** → connect to your GitHub repo
4. Pick build settings: **publish directory: `/`**, no build command needed for plain HTML
5. Deploy

After this, every time you commit + push to GitHub, Netlify auto-rebuilds in ~30 seconds. You can ask Claude Code to make changes locally, then "push to GitHub" — and your live site updates.

### Add to Home Screen (mobile)

On your phone:
1. Open the URL in Safari (iPhone) or Chrome (Android)
2. Share menu → **Add to Home Screen**
3. Name it, tap Add

It launches like a native app, with full screen and no browser chrome. This single step makes the app feel real.

### Custom domain (optional, ~$10/year)

If you want `myapp.com` instead of `myapp.netlify.app`: buy a domain at **Cloudflare Registrar** (cheapest) or **Namecheap**, then in Netlify: **Domain management** → **Add custom domain**. Netlify walks you through the DNS records.

---

## Phase 4 — (Optional) Cross-device sync via Supabase

**Skip this phase if** you only use the app on one device. **Add it when** any of these is true:

- You want phone AND laptop to share the same checklist/state
- More than one person will use the app
- You'd be sad if your localStorage got cleared by accident

**Goal:** Replace `localStorage` with a Supabase-backed sync layer that updates in realtime across devices.

### Set up Supabase

1. Sign up at **supabase.com** (use Sign in with GitHub if you have one)
2. Create a new project. Pick the region closest to your users. Save the database password in your password manager.
3. Wait ~5 minutes for the project to provision.

### Enable Anonymous Sign-Ins (do this now — Phase 5 needs it)

In Supabase: **Authentication** → **Sign In / Up** → scroll to **User Signups** → toggle **Allow anonymous sign-ins** ON → **Save**.

### Get your project credentials

Project Settings (gear icon, bottom of sidebar) → **API**. You need two values:

- **Project URL** — looks like `https://xxxxxxxx.supabase.co`
- **anon public key** — long JWT-looking string starting with `eyJ...`

**Important:** there's also a `service_role` key on this page. *Never* paste that anywhere. Bypasses all security. The `anon` key is designed to be public and ship in client-side code.

### The schema migration prompt

Now hand both values to Claude Code, plus a description of your data model:

> I want to add Supabase-backed cross-device sync to this app. Here's what I want:
>
> - State currently lives in `localStorage` under the key `[your-key]`. The shape is `[paste a sample of your data]`.
> - This will be a shared workspace among `[N]` users.
> - Use Supabase anonymous sign-ins (already enabled). Auth flow will come in Phase 5; for now, just wire up the data layer.
> - Use realtime subscriptions so changes propagate across devices in ~1 second.
> - Existing localStorage data should auto-migrate to Supabase on first sign-in.
>
> Project URL: `[paste]`
> anon key: `[paste]`
>
> Write the SQL migration first, then update the HTML.

### Critical: avoid recursive RLS policies

Row-Level Security (RLS) is how Supabase enforces "each user only sees their own data." When Claude Code writes RLS policies for the **member table** (the table that says who has access to what), make sure the policy is **non-recursive**.

✅ **Correct:** `using (user_id = auth.uid())`

❌ **Wrong:** `using (id IN (SELECT id FROM members WHERE user_id = auth.uid()))` — this references the same table the policy protects, causing infinite recursion. Postgres will return error code **42P17** at runtime.

If you see error 500 with `proxy_status: "PostgREST; error=42P17"` in the Supabase API logs, this is the cause.

### Realtime: enable it on data tables

Realtime subscriptions don't work unless the relevant tables are added to the publication:

```sql
alter publication supabase_realtime add table public.[your_data_table];
```

Claude Code should include this in the migration. Verify it's there.

---

## Phase 5 — (Optional) Authentication: shared passcode

**Goal:** Gate access to the app behind a passcode you share with your group. No emails, no accounts to manage.

### Why we don't recommend email magic links for personal apps

Three real problems:

1. **Supabase free-tier email service is rate-limited to 2 emails per hour, per project.** Trips during testing, leaves users locked out for an hour. Sometimes more if attempts keep stacking.
2. **Custom SMTP relays (Resend, Mailgun, SendGrid, Brevo) all require domain ownership now.** Google/Yahoo/Microsoft tightened sender authentication rules in 2024; transactional senders enforce DKIM/DMARC compliance, which requires DNS records on a domain *you own*.
3. **Personal email SMTP (Gmail, Outlook) works** but ties the app to a specific personal account, sender appears as your personal email, and the path is being gradually deprecated.

For 2–5 users, the value of "each person has their own account" doesn't justify the operational cost.

### The shared-passcode model

- One bcrypt-hashed passcode in Supabase
- App shows a passcode prompt on first load
- On submit: client signs in **anonymously** (creates a real auth session with no identity), then calls a Postgres function that verifies the passcode and adds the anonymous user to the access list
- Session persists; users only enter the passcode once per device

Tradeoff: no per-user attribution. For a 3-person family planner, this is fine. For anything where you'd want "Kyle ticked this" history, use email auth + buy a domain.

### The implementation prompt

> Switch this app's auth from `[whatever it currently uses]` to shared-passcode + Supabase anonymous sign-ins.
>
> Write a SQL migration that:
> 1. Adds an `[your_app]_config` table with a `passcode_hash text` column (bcrypt, via pgcrypto)
> 2. Creates a Postgres function `join_[your_app]_with_passcode(input_passcode text) returns boolean`, security definer, that verifies the passcode using `extensions.crypt()` and adds the caller (`auth.uid()`) to the members table
> 3. Grants execute on that function to `anon` and `authenticated`
> 4. Removes any prior email-invite mechanism
> 5. Sets the initial passcode (use a placeholder I'll replace with my real one)
>
> Update the HTML so the sign-in screen takes a passcode (`type="password"`), calls `signInAnonymously()` then the join function, handles wrong-passcode gracefully (sign out the orphan anon user), and gates the app on membership.

### Setting and rotating the passcode

The passcode is bcrypt-hashed. You set it via SQL:

```sql
update public.[your_app]_config
set passcode_hash = extensions.crypt('your-passcode-here', extensions.gen_salt('bf'));
```

To rotate: run the above with a new value. Existing devices stay unlocked (their session is still valid). To force everyone to re-unlock with the new code, also run:

```sql
delete from public.[your_app]_members;
```

### How to share the passcode

Verbally, or via a secure messaging app. Don't email it. Don't put it in the URL. Don't write it in the app description on a public site.

---

## Phase 6 — Verify and roll out

**Goal:** Confirm the app works end-to-end before declaring victory.

### Have Claude Code verify with Playwright

> Verify the deployed app end-to-end with Playwright:
> - Passcode prompt renders, accepts input
> - Wrong passcode shows an error and doesn't leak access
> - Correct passcode unlocks the app
> - Ticking a checkbox persists across reload
> - Open in two contexts simultaneously: changes in one appear in the other within ~2 seconds

If anything fails, give Claude Code the failing test output and iterate.

### Personally test the full flow

1. Open your URL in a fresh private/incognito window
2. Enter the passcode
3. Use the app for 5 minutes — tick things, type a note, navigate around
4. Open it on your phone, sign in with the same passcode
5. Tick something on the laptop, watch it appear on the phone
6. Add to Home Screen on the phone

### Roll out to your group

Send each person:
- The URL
- The passcode (verbally or via secure DM)
- A line about Add-to-Home-Screen

That's it. They open the URL, enter the code, and they're in.

---

## Common pitfalls — one-page reference

**RLS infinite recursion (`error=42P17`).** Member-table policies must use `user_id = auth.uid()` directly, not subqueries against the same table. Phase 4.

**Email rate limits hit instantly.** Free Supabase auth-email is 2/hour. Don't try to debug — switch to passcode auth (Phase 5).

**SMTP relays reject your sender as "non-compliant."** Brevo, Resend, SendGrid all require domain ownership. If you don't have a domain, don't try them. Either use Gmail/Outlook SMTP (uses your personal account) or skip email auth entirely.

**Brevo suspends your account before you've sent anything.** Their anti-fraud is aggressive on new free accounts. There's no fast appeal. Don't invest time in Brevo unless you already have an established account.

**Magic-link redirects to a blank page.** The `redirect_to` URL must be in Supabase's allow list (Authentication → URL Configuration). Add `https://yourapp.netlify.app/**` (the `**` wildcard is important).

**Tailwind CDN + Babel render timing trips Playwright.** When testing, wait for `networkidle` AND a visible content selector before asserting. The in-browser JSX compile takes 1–2 seconds.

**The `service_role` key gets shipped to the client.** This is a disaster — it bypasses all RLS. Only the `anon` key is safe in client code. If you ever paste the service role key into client-side code, **rotate it immediately** in Supabase.

**Anonymous sign-ins return "disabled."** Toggle them on in Authentication → Sign In / Up → User Signups → Allow anonymous sign-ins.

**Realtime subscriptions silently don't fire.** Tables must be added to the `supabase_realtime` publication: `alter publication supabase_realtime add table public.your_table;`

---

## Prompts cheat sheet

Paste-ready prompts for each phase. Replace bracketed placeholders.

### Claude Design — opening prompt

> I want to build a `[type of app]` for `[purpose]`.
>
> Aesthetic: `[mood, colors, typography vibe, references]`.
>
> Core features:
> - `[feature]`
> - `[feature]`
> - `[feature]`
>
> Persist state via `window.storage`. `[Locale + constraints]`.
>
> Start with a single-column immersive design and we'll iterate.

### Claude Design — audit loop

> Can we audit this as a QC to see if there's anything we need to edit or update or make more user-friendly within the UI/UX?

Then: *"Apply fixes 1, 2, 4."* (or *"all of them"*)

### Claude Code — handoff

> Fetch this design file, read its readme, and implement the relevant aspects of the design. `[handoff URL]`
>
> Implement: `[filename from bundle]`

### Claude Code — strip design affordances (if needed)

> Strip the design-tool affordances from this file: TweaksPanel, postMessage edit-mode hooks, EDITMODE markers, and `window.storage`. Replace `window.storage` with `localStorage`. Bake in the chosen tweak defaults as constants.

### Claude Code — verify

> Verify this renders and behaves correctly in a browser using Playwright. Test: `[2–3 specific interactions]`.

### Claude Code — add Supabase sync

> Add Supabase-backed cross-device sync. State is in localStorage under key `[key]`, shape: `[paste sample]`. `[N]` users, shared workspace. Use anonymous sign-ins. Realtime subscriptions for ~1s propagation. Migrate existing localStorage on first sign-in.
>
> Project URL: `[paste]`
> anon key: `[paste]`
>
> Write the SQL migration first, then update the HTML.

### Claude Code — switch to passcode auth

> Switch the auth model to shared passcode + Supabase anonymous sign-ins. Bcrypt-hashed passcode stored in `[app]_config` table. Postgres function `join_[app]_with_passcode(input_passcode text) returns boolean` (security definer) verifies and adds caller to members. Grant execute to `anon` and `authenticated`. Remove any prior email-invite mechanism. Update HTML with passcode prompt that uses `signInAnonymously()` + the join function.

### Setting / rotating the passcode (run in Supabase SQL Editor)

```sql
update public.[your_app]_config
set passcode_hash = extensions.crypt('your-passcode-here', extensions.gen_salt('bf'));
```

To force everyone to re-unlock:

```sql
delete from public.[your_app]_members;
```

---

## Closing note

The pattern in this tutorial — design with one Claude tool, implement with another, deploy somewhere free, optionally add a tiny backend — is a **good fit for personal tools**: family planners, small group trackers, learning aids, journaling apps. It's a **bad fit** for anything you'd sell or ship to strangers — those need real auth, abuse protection, support, monitoring, billing, lawyers.

Know which one you're building. If it's the first kind, the path above gets you to a real, usable, polished thing in an afternoon, with no code written by you.
