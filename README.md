# Marko's Birthday Party - Squad Invite System

A gaming-themed birthday party invite site with RSVP tracking, a live bulletin board, and email notifications - all backed by Google Sheets and Google Apps Script.

---

## System Architecture

```
Google Sheet (Invitees + BulletinBoard tabs)
        ↕
Google Apps Script (code.gs) - deployed as Web App
        ↕
GitHub Pages (index.html) - rtheman.github.io/marko-party
        ↕
Guest's browser (personal invite link)
```

**Files at a glance:**

| File | Where it lives | Purpose |
|---|---|---|
| `index.html` | GitHub Pages | The invite website guests visit |
| `code.gs` | Google Apps Script | Backend API (RSVP, bulletin, email) |
| `marko birthday party data.gsheet` | Google Drive | Guest list + bulletin data |
| `marko birthday party invite.jpeg` | Google Drive | Party invite image (reference only) |

---

## Logic Flow

### Sending Invites
1. Admin adds guest name + email to the `Invitees` sheet
2. Admin adds guest to the `GUESTS` object in `index.html` and redeploys to GitHub Pages
3. Admin runs `sendInvites()` in the Apps Script editor - each guest receives a personalised email with their unique link (e.g. `rtheman.github.io/marko-party/?guest=theirname`)

### Guest RSVP
1. Guest clicks their personal link → lands on the site with their name pre-filled
2. Guest clicks **"I'M IN - JOIN SQUAD"** or **"Can't make it"**
3. Site sends a POST request to the Apps Script web app
4. Apps Script records `CONFIRMED` or `DECLINED` in the `RSVP_status` column (col F) of the `Invitees` sheet

### Bulletin Board
1. Admin opens the site, clicks **"POST SQUAD UPDATE"**, enters the organiser password (`Marko12`)
2. Post is saved locally and synced to the `BulletinBoard` sheet via Apps Script
3. Apps Script emails all `CONFIRMED` guests with the update

---

## Key Configuration Values

| Setting | Value / Location |
|---|---|
| Google Sheet ID | `1uDXrNnlw92vDChv-wCWJ8lUTbpSrQkfTHolD39-Qxxs` |
| Apps Script project | [script.google.com - Marko Birthday Party](https://script.google.com/home/projects/1c-isLfESj4ATZAV-y21aW5SIYJPrpEw5BZ4bRsaZnzXTmKQ0b-Uu-_Oq/edit) |
| Apps Script web app URL | Set in `index.html` → `CFG.appsScriptUrl` |
| GitHub Pages URL | `https://rtheman.github.io/marko-party/` |
| Organiser password | `Marko12` (bulletin board only) |

---

## Admin Playbook

### Add a new guest

1. **Google Sheet** (`Invitees` tab): add a new row with the guest's name (col A) and email (col C)
2. **`index.html`** (`GUESTS` object, around line 895): add an entry:
   ```js
   const GUESTS = {
     'henric':      'Henric',
     'newguestid':  'New Guest Name',  // ← add here
   };
   ```
   The key must be the guest's name **lowercased with spaces removed** (e.g. "John Smith" → `johnsmith`)
3. **Commit and push** `index.html` to GitHub - GitHub Pages updates within ~1 minute
4. **Send invite**: in the Apps Script editor, select `sendInvites` from the function dropdown and click **Run** - it will send to any guest not yet marked `Invite_sent = TRUE`

---

### Remove or uninvite a guest

1. Delete (or clear) their row in the `Invitees` sheet
2. Remove their entry from the `GUESTS` object in `index.html`
3. Commit and push `index.html`

Their personal link will stop working immediately after the push.

---

### Change a guest's RSVP manually

1. Open the `Invitees` sheet
2. Find the guest's row
3. Set col F (`RSVP_status`) to `CONFIRMED` or `DECLINED`

---

### Post a bulletin board update

1. Go to `rtheman.github.io/marko-party/`
2. Scroll to **Squad Bulletin Board** → click **POST SQUAD UPDATE**
3. Enter password: `Marko12`
4. Type your message (and optionally paste an image URL)
5. Click **TRANSMIT TO SQUAD** - all `CONFIRMED` guests receive an email notification

---

### Update the Apps Script backend (code.gs)

1. Open the [Apps Script project](https://script.google.com/home/projects/1c-isLfESj4ATZAV-y21aW5SIYJPrpEw5BZ4bRsaZnzXTmKQ0b-Uu-_Oq/edit)
2. Edit the code
3. If you changed `doGet` / `doPost` behaviour: **Deploy → Manage Deployments → edit the existing deployment → new version → Deploy**
4. If you only changed helper functions like `sendInvites`: no redeployment needed, just save and run

---

### Resend invites (e.g. after adding new guests)

`sendInvites()` skips any row where `Invite_sent = TRUE`, so it's safe to re-run at any time. It will only email guests who haven't been invited yet.

To force-resend to a specific guest: clear their `Invite_sent` cell in the sheet, then re-run `sendInvites()`.

---

### Deploy index.html changes to GitHub Pages

```bash
# from the project directory
git add index.html
git commit -m "your message"
git push origin main
```

GitHub Pages serves from the `main` branch root. Changes go live within ~60 seconds.

---

## Invitees Sheet - Column Reference

| Col | Header | Notes |
|---|---|---|
| A | Participant | Guest's display name (must match `GUESTS` key) |
| B | *(spare)* | |
| C | email | Used for invite + notification emails |
| D | *(spare)* | |
| E | Invite_sent | Set to `TRUE` by `sendInvites()` automatically |
| F | RSVP_status | `CONFIRMED` / `DECLINED` - set by RSVP flow or manually |
