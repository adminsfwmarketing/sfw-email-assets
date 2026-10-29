# Build Plan: SFW Email Builds (HTML → Campaign Monitor)

*Status as of Oct 28, 2026. Upload this to the Team project as a project file and keep it updated as each step is finished.*

## The goal

Cut the time between "copy approved" and "draft ready in Campaign Monitor." Claude writes the email, builds it, hosts the assets and creates the draft campaign in the right client account. Danielle or Emilie then reviews it, tests it and schedules it.

## How it works

```
Approved copy + images
        │
        ▼
Claude builds the email (template content or full HTML)
        │
        ▼
Images / HTML uploaded to the PUBLIC EMAIL HOST (GitHub Pages)
        │
        ▼
Zapier → Campaign Monitor API → DRAFT campaign in the client's account
        │
        ▼
Danielle / Emilie: review → test send → schedule
```

**Why we need a host:** the Campaign Monitor API won't accept pasted HTML or uploaded image files. It fetches them from a public web address, and once imported, Campaign Monitor keeps its own copy. So the host only has to work when the import happens.

**Why we're building templates:** once a client has a Campaign Monitor template with editable regions (header and footer locked, body editable), each new campaign only needs text and image links. That means no HTML hosting per email, and you can still edit drafts in Campaign Monitor's own editor.

---

## What's already in place

| Item | Status |
|---|---|
| Zapier ↔ Campaign Monitor (Danielle's Zapier account) | ✅ Working. Tested Oct 8 on the DAU account (twice). |
| Client connections | ✅ LEW, DAU, GAR, GIB, WAN, MOR, YAN (Advisor Assist) |
| | ⚠️ One unnamed connection has an expired API key. Reconnect or delete it. |
| | ❌ Not connected: **FOX, HIL, MAC**. RET won't be connected for now (out of scope for this build process). |
| Email-safe HTML pattern | ✅ Built and tested in a browser: the FOX Thanksgiving e-card, with branded header and footer, 600px wide and mobile rules |
| Canva connection | ✅ Working for header images |
| WordPress as host | ❌ Ruled out. Disconnected for security reasons. |
| Dropbox as image host | ❌ Ruled out Oct 28. Campaign Monitor won't download images from Dropbox links (see Decision 5). |

---

## Decisions

1. **Public host: ✅ GitHub Pages** (decided Oct 8). Free, keeps a history of every file, and Claude can upload with an access token. Everything on it is public. Only email images and HTML go there.

2. **How Emilie gets access to Campaign Monitor: ⏸ Deferred until after the pilot.** Zapier connections belong to the person who made them, so Emilie can't use Danielle's. Options:
   - **(a)** Emilie connects her own Zapier account to each client's Campaign Monitor account.
   - **(b)** Danielle shares her Campaign Monitor connections with Emilie through a Zapier team plan.
   - **(c)** Emilie builds the content and Danielle runs the "create draft" step.

3. **Pilot client: ✅ DAU** (decided Oct 8). It's connected, it was tested Oct 8, and its ClientID is on file.

4. **Default font: ✅ Arial for all newsletters** (decided Oct 28 by Danielle). Web fonts like Nunito only show in Apple Mail and Outlook for Mac. Outlook on Windows, Outlook.com and Gmail fall back to Arial anyway. Use `font-family: Arial, Helvetica, sans-serif` everywhere and don't load Google Fonts in email HTML.

5. **Image hosting: ✅ GitHub Pages only** (Oct 28). Campaign Monitor refuses Dropbox image links (both `www.dropbox.com …raw=1` and `dl.dropboxusercontent.com`): the import saves empty images, or the API returns error 321 ("couldn't download"). A Dropbox `raw=1` link *does* work as the `HtmlUrl` for the email HTML itself. Until Claude can push to the repo, Danielle uploads images through GitHub web. If a commit fails when replacing a file, reload the page (expired session) or upload under a new name (e.g. `-v2`).

---

## Build checklist

### Phase 1: Set up the host (about 1 hour)
- [x] Choose the host: GitHub Pages.
- [x] Create the account and storage space: repo `adminsfwmarketing/sfw-email-assets`, Pages on (Oct 8).
- [x] Install the Claude GitHub app with access to that repo only (Oct 8). Claude uploads through this, no token needed. Note: a session only gets repo access if it starts after the app is connected **and the repo is selected as a source for that session**.
- [x] ~~Access token~~ Not needed. Replaced by the Claude GitHub app. Delete the Oct 8 "Claude email uploads" token if it still exists.
- [ ] Test: Claude uploads one image, confirm the public link loads in a private browser window, then delete it.
  - Oct 8: a test.png was committed at 2:19 pm and deleted at 2:20 pm from GitHub web, but no one recorded that the public link loaded. Pages root returns 200.
  - ⚠️ Still blocked Oct 28: Claude still can't push ("GitHub access to this repository is not enabled for this session"). Fix: start a Claude session with `adminsfwmarketing/sfw-email-assets` added as a source. Workaround: Danielle uploads files through GitHub web (Add file → Upload files). New files take about a minute to go live on Pages.
- [x] Base URL: `https://adminsfwmarketing.github.io/sfw-email-assets/` (loads, confirmed Oct 8). Folder convention: `/<CODE>/<YYYY-MM>/<campaign-slug>/file.png`. The repo is synced in Danielle's Dropbox at `Dropbox\GitHub\sfw-email-assets`; Claude saves files there, and Danielle uploads them to GitHub.

### Phase 2: Pilot client template, DAU (about 2–3 hours)
- [x] Get DAU's current header and footer. (Oct 22: Danielle supplied a Beefree export, "Daumler Header and Footer.zip".)
  - Oct 8: account check ✅ (GET /clients.json → "Daumler, Mertz & Associates"). DAU has **no saved templates** in CM. 88 sent campaigns, 17 drafts.
  - Reference emails: Back to School Webinar 2026 (sent Sep 1, 416 recipients, CampaignID ba48e491f138b958bb645f3ebb532ba7) and April NL (draft, CampaignID 5d39bfae05cf57f92f9322ccc49a88f9).
- [x] Get list IDs, and from name and email through the API. Add them to the Client Registry. (Oct 8, main list to confirm)
- [x] Build the Campaign Monitor template (Oct 22, v1). Header and footer are locked. One repeater with 8 blocks: hero image, headline, body text, button, image + text side by side, inset image with caption, divider, sign-off. It also has a preheader field and the CM `<unsubscribe>` tag. Previewed on desktop and at 375px mobile.
  - Files: `DAU/2026-10/dau-template-v1/` → dau-template.html, dau-logo-header.png, dau-logo-footer.png (logos resized to 2× retina), 3 social icons, 3 placeholder images.
  - Template HTML URL: `https://adminsfwmarketing.github.io/sfw-email-assets/DAU/2026-10/dau-template-v1/dau-template.html`
  - Button colour was #0071CE. Switch to IG navy #001E60 to match the fireside invite.
- [x] Upload the `DAU/` folder to the repo and create the template. Done by Oct 28: "DAU Newsletter v1", TemplateID `ae80d3f022c9b8c1f7b6ba679232623c`.
- [ ] Template update pending (Oct 28): `dau-template.html` in the Dropbox repo folder is now Arial (Decision 4). Upload it to GitHub, then Claude updates the CM template with `PUT /templates/ae80d3f022c9b8c1f7b6ba679232623c.json`. Also change its button colour to #001E60.
- [x] Open the template in Campaign Monitor and test the editor (Oct 28). Text editing worked, but Danielle couldn't add images in the editor. For custom emails, use the Claude editing page instead (see Phase 3).

### Phase 3: Pilot campaign (about 1 hour)
- [x] Use a real upcoming DAU email: the Fireside Evening invitation (event Nov 19, 2026, Calgary Golf & Country Club).
- [x] Claude creates the draft. Done Oct 28 via the HTML route: "DAU — Fireside Evening Invitation — 2026-10-28" (CampaignID 7e3d78f4f004e9cc9fe09f85a976b4b3), all images confirmed loading. Recipients are set to the Team Emails placeholder; swap in the invite list.
  - Editing workflow that worked: Danielle edits on a private Claude editing page ("Fireside Invite Editor" artifact) and clicks Save; Claude reads the saved version, cleans the HTML and creates a fresh draft.
  - Two older fireside drafts (broken images, older copy) are still in DAU Drafts and should be deleted.
- [ ] Review in Campaign Monitor, test send to Danielle and Emilie, and check on mobile and in Outlook.
- [ ] Note what needed fixing by hand, and roughly how much time it saved.

### Phase 4: Roll out
- [ ] Connect FOX, HIL and MAC in Zapier, if they use Campaign Monitor. (RET won't be connected for now.)
- [ ] Reconnect or delete the expired unnamed connection.
- [ ] Build a template for each remaining client, about 1–2 hours each. Start with whoever sends most often. Use Arial (Decision 4).
- [ ] Set up Emilie's access (Decision 2) and have her run one full email herself.

### Phase 5: Optional later additions
- [ ] Monthly open and click summaries per client, pulled from Campaign Monitor reports.
- [ ] A seasonal library of reusable header images: Thanksgiving, holidays, RRSP season, tax season.

---

## Client Registry (starter: fill in as we go)

| Code | Client | Zapier connection | CM ClientID | TemplateID | Main list ID | From name / email | Footer on file |
|---|---|---|---|---|---|---|---|
| GIB | Gibbings Aylward Group | GIBBINGS Campaign Monitor | | | | | |
| MAC | Kevin MacKay & Matthew Bailey | — not connected | | | | | |
| DAU | Daumler, Mertz & Associates | DAUMLER Campaign Monitor (account check ✅ Oct 8, Oct 28) | 233e65c98f5fd0457ebbba8f41ce2423 | ae80d3f022c9b8c1f7b6ba679232623c ("DAU Newsletter v1"; Arial update pending) | d30921110f8db5123303380c4f410faf (⚠️ named "Daumler Mertz Clients with Birthdays August 2026", ~415 active) + Team Emails e44782e5998b1dc83922e2d499f353e9 | Daumler, Mertz & Associates / daumler.associates@myig.ca, reply-to daumler.associates@igpwm.ca | ✅ Oct 22 (below) |
| FOX | Fox-Revett Heslegrave & Associates | — not connected | | | | | ✅ from Thanksgiving build |
| GAR | Garofalo & Associates | GAROFALO Campaign Monitor | | | | | |
| WAN | Wanner Nair & Associates | WANNER Campaign Monitor | | | | | |
| HIL | Hillaby & Associates | — not connected | | | | | |
| MOR | Morris & Morris | MORRIS Campaign Monitor | | | | | |
| LEW | Llewellyn & Associates | LLEWELLYN Campaign Monitor | | | | | |
| RET | Rettinger Group | — won't be connected for now | | | | | |
| YAN | Brandon Yanchus | Advisor Assist Campaign Monitor | | | | | |

**DAU notes (Oct 8):** The main client list is misnamed "Daumler Mertz Clients with Birthdays August 2026". It's what the Sep 1 Back to School send used (416 recipients). "Newsletter Subscriptions" has 0 subscribers. The Feb 2026 newsletter went to "Client Emails Updated January 2026" (b18cd6b618c2d54eca61aefd789ed410), which no longer shows in the active lists. Confirm the main list with Danielle, and consider renaming it in CM. Newsletter cadence: monthly NL plus market updates and webinar invites. Last full-list send was Sep 1; Comedy Night sends (Aug–Sep) went to small individual lists. There's an old draft named "NOT READY TO SEND Gordie Howe CARES Tournament May 22-24 2026" sitting in Drafts. Leave it alone.

**DAU footer (Oct 22, from Danielle's header/footer export):** Daumler, Mertz & Associates, Private Wealth Management · 225 6th Ave SW Suite 900, Calgary, AB T2P 1N2 · 403-338-2255 · daumler.associates@igpwm.ca · "IG Wealth Management Inc." · Unsubscribe · Privacy (https://www.ig.ca/en/legal/privacy) · "Trademarks including IG Wealth Management, are owned by IGM Financial Inc. and licensed to its subsidiary corporations." · "Copyright 2026 © Daumler, Mertz & Associates Private Wealth Management. All rights reserved." Social: Facebook /RobertKDaumler, LinkedIn /in/robertdaumler, YouTube channel UCEi9vDbTtT3vUJmcMoJJwZA. Header menu: About Us (daumlermertzwealth.ca), Client Login (secure.ig.ca), Contact Us (mailto). Colours: top bar #1E1A34, menu/headings #122b49, footer #0e1631, logo blue #0071CE, IG Private Wealth navy #001E60 (from the IG logo; replaces the mid blue for buttons, labels and accents as of Oct 28). Font: Arial (Decision 4); previously Nunito with Arial fallback.
⚠️ The DAU footer says "IG Wealth Management Inc." but the FOX footer says "IG Wealth Management Inc. Mutual Fund Division". Confirm which disclaimer is current. ⚠️ The copyright year is hard-coded, so update it in January.

**FOX footer (from the Thanksgiving build):** Fox-Revett Heslegrave & Associates, Private Wealth Management · Suite 10A, 17310 Yonge Street, Newmarket ON L3Y 7R9 · Fox-Revett.Heslegrave@igpwm.ca · 905-952-2029 · IG Wealth Management Inc. Mutual Fund Division · "Trademarks including IG Wealth Management, are owned by IGM Financial Inc. and licensed to its subsidiary corporations." · Copyright line. Header brand colours: lavender #E4DFF2, top bar #1E1A34, link blue #2B71B9, footer #251C45.
⚠️ The firm name differs between the header logo ("Fox-Revett & Associates") and the footer ("Fox-Revett Heslegrave & Associates"). Confirm which to use.

---

## What to move into the Team project

1. These project instructions, pasted into the instructions field
2. This Build Plan, uploaded as a file
3. The SFW Brand Kit, so drafts follow the house voice
4. The client list CSV
5. The FOX Thanksgiving HTML files, as the reference build for the email pattern
