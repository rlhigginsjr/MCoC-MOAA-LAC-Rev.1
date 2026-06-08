# MOAA Legislative Action Center — Version 2

A web-based advocacy tool that makes it simple for MOAA members to contact their elected officials on priority state and federal legislation — directly from their own email client, with no backend infrastructure, no subscriptions, and no technical expertise required by members.

**Live demo:** [rlhigginsjr.github.io/LAC](https://rlhigginsjr.github.io/LAC/)

Developed by the **Michigan Council of Chapters, MOAA (MCOC)** and offered freely to all state MOAA Councils.

---

## What's New in Version 2

- **State / Federal tabs** — State and federal legislation are organized in separate tabs; members toggle between them with one click
- **Automatic legislator lookup** — Bills targeting a member's own district use the Google Civic Information API to find their specific state rep and state senator from their address and ZIP code automatically
- **Three targeting modes** — Each bill specifies how legislators are selected: fixed committee, member's district, or full delegation
- **Federal tab** — Fully built and ready; displays a "Coming Soon" placeholder until federal bills are added; tab badge updates automatically when bills are loaded
- **Cross-linking** — State bill cards can link to related federal bills and vice versa
- **Graceful fallback** — If the legislator lookup fails, a clear message and manual lookup link are shown

---

## How It Works

Members complete three steps:

1. **Enter their information** — name, address, city, ZIP, email, and MOAA chapter
2. **Select a bill** — from the current priority legislation on either the State or Federal tab
3. **Send their message** — a personalized email opens pre-filled in the member's own email client, one button per legislator, ready to review and send

No data is collected. No accounts are required. Nothing is stored.

---

## Technical Overview

- Single self-contained HTML file — no frameworks, no dependencies, no build step
- Google Civic Information API provides automatic legislator lookup by address (free, no usage limits at MOAA volumes)
- API key authentication — no service account, no OAuth, no server-side code required
- Email merge fields replaced at runtime with member-entered data
- Each legislator button generates a `mailto:` link with subject and body pre-populated
- Personalized phone call script generated automatically at Step 3
- Hosted free on GitHub Pages; links from any website with a button or one iframe line

---

## Before Deploying — API Key Setup

Version 2 requires a free Google Civic Information API key for the legislator lookup feature. To set one up:

1. Go to [console.cloud.google.com](https://console.cloud.google.com) and sign in with any Google account
2. Create a new project (name it anything, e.g. `MOAA-LAC`)
3. Go to **APIs & Services → Library**, search `Civic Information`, and click **Enable**
4. Go to **APIs & Services → Credentials → + Create Credentials → API key**
5. Restrict the key: set **HTTP referrers** to your GitHub Pages URL and your council website URL (e.g. `youraccount.github.io/*` and `www.yourcouncil.org/*`)
6. Set **API restriction** to **Google Civic Information API**
7. Save the key

Open `index.html` in a plain text editor, find this line near the top of the script section:

```javascript
const GOOGLE_API_KEY = 'YOUR_API_KEY_HERE';
```

Replace `YOUR_API_KEY_HERE` with your key, save, and upload to GitHub.

> **Security note:** Never paste your API key into a chat, email, or public document. The domain restrictions you set in Google Cloud Console are your primary protection — the key will not work from any other website.

---

## Using This Code for Your State Council

The source code is free to download and use. There are two paths:

### Path 1 — Self-Service (Free)

Download `index.html`, adapt it for your state's legislation and legislators, obtain your own Google Civic Information API key, host on GitHub Pages, and link from your council's website. Requirements:

- Ability to edit a text file
- A free GitHub account
- A free Google Cloud account for the API key
- Access to your council's website to add a button or embed link

No step-by-step deployment guide is provided beyond this README. Councils with a technically capable volunteer should be able to work from the source code directly.

### Path 2 — Guided Deployment ($1,000 contribution)

MCOC will handle the complete deployment for your council — researching and populating your priority legislation, verifying legislator contact information, configuring the API key, applying your council's branding, setting up GitHub Pages hosting, linking from your website, and conducting a training session so your team can maintain it going forward.

In lieu of a service fee, councils are asked to make a **$1,000 tax-deductible contribution** to the **Southeastern Michigan Chapter, MOAA Scholarship Fund**, which supports educational scholarships for Michigan military families.

To request a guided deployment, contact:

**LCDR Rich Higgins, USN (Ret.)**
President, Michigan Council of Chapters, MOAA
[moaamcoc.com](https://www.moaamcoc.com)

---

## Bill Targeting Modes

Each bill in the system has a `target` field that controls which legislators receive the email:

| Mode | How it works | When to use |
|------|-------------|-------------|
| `committee` | Fixed list of legislators defined in `STATE_COMMITTEE` or a bill-level committee array | Bills referred to a specific committee |
| `district` | Google Civic API looks up member's own state rep and state senator by address | Bills targeting the full chamber or member's personal delegation |
| `delegation` | Michigan's 2 U.S. Senators + member's U.S. House rep (federal tab only) | Federal bills targeting the full Michigan congressional delegation |

---

## Maintaining Your Deployment

### Adding a State Bill — `const STATE_BILLS = [` in the script section

Each bill is one entry `{ ... }` inside the array. Use this template:

```javascript
  {
    id: 5,
    code: 'HB 1234',
    priority: false,
    target: 'committee',
    short: 'One-line title of the bill',
    desc: 'One or two sentences describing what the bill does.',
    url: 'https://www.legislature.mi.gov/link-to-bill-text.pdf',
    relatedFederal: null,
    subject: 'Support for HB 1234 — Short description',
    body: `Dear [SALUTATION] [LAST_NAME],

Your advocacy email text goes here.

Respectfully,
[FULL_NAME]
[CITY], Michigan
Member, [CHAPTER]`
  },
```

To link a state bill to a related federal bill, set `relatedFederal` to the federal bill code:
```javascript
relatedFederal: 'S.1234',
```

### Adding a Federal Bill — `const FEDERAL_BILLS = [` in the script section

Same structure as state bills with two differences — use `target: 'delegation'` for bills targeting the full Michigan congressional delegation, and set `relatedState` instead of `relatedFederal`:

```javascript
  {
    id: 1,
    code: 'S.1234',
    priority: true,
    target: 'delegation',
    short: 'One-line title of the bill',
    desc: 'One or two sentences describing what the bill does.',
    url: 'https://www.congress.gov/bill/...',
    relatedState: 'HB 5280',
    subject: 'Please support S.1234',
    body: `Dear [SALUTATION] [LAST_NAME],

Your advocacy email text goes here.

Respectfully,
[FULL_NAME]
[CITY], Michigan
Member, [CHAPTER]`
  },
```

The Federal tab badge updates from "Coming Soon" to a bill count automatically when entries are added.

### Bill Fields Reference

| Field | Purpose |
|-------|---------|
| `id` | Unique number — increment for each new bill |
| `code` | Bill number, e.g. `'HB 1234'` or `'S.99'` |
| `priority` | `true` shows the gold Primary target badge |
| `target` | `'committee'`, `'district'`, or `'delegation'` |
| `short` | Brief title shown on the bill card |
| `desc` | One or two sentence plain-English description |
| `url` | Link to the full bill text |
| `relatedFederal` | Federal bill code for cross-link, or `null` |
| `relatedState` | State bill code for cross-link (federal bills), or `null` |
| `subject` | Email subject line |
| `body` | Full email template — use merge fields below |

### Merge Fields

| Field | Replaced with |
|-------|--------------|
| `[FULL_NAME]` | Member's full name |
| `[LAST_NAME]` | Legislator's last name |
| `[CITY]` | Member's city |
| `[ZIP]` | Member's ZIP code |
| `[EMAIL]` | Member's email address |
| `[ADDRESS]` | Member's street address |
| `[CHAPTER]` | Member's MOAA chapter |
| `[SALUTATION]` | Representative or Senator |

**Critical:** Every bill entry except the last must end with `},` (with the comma). The final entry ends with `}` before the closing `];`. A missing or extra comma will cause the page to go blank.

### Updating Committee Members — `const STATE_COMMITTEE = [`

Update when committee membership changes. Each entry:

```javascript
{ 
  name: 'Full Name',
  title: 'Role · Party · District',
  salutation: 'Representative',
  email: 'email@house.mi.gov',
  phone: '(517) 000-0000'
},
```

### Updating Michigan U.S. Senators — `const MI_SENATORS = [`

Update when Senate seats change. Same structure as committee members — these are pre-loaded for all federal delegation-targeted bills.

---

## Michigan Deployment — Current Legislation

| Bill | Title | Target | Status |
|------|-------|--------|--------|
| **HB 5280** ★ | Income Tax Act — retirement pay equity for NOAA & USPHS officers | Committee | **Primary target** |
| HB 5262 | Uniformity of Service Dates Act — veteran recognition | Committee | Active |
| HB 5278 | State Personal Identification Card Act — veteran ID designation | Committee | Active |
| HB 5279 | Michigan Vehicle Code — veteran driver's license designation | Committee | Active |

Targeting the **Michigan House Committee on Government Operations** — five members pre-loaded with verified email addresses and phone numbers.

Federal bills: none yet — tab is live and ready.

---

## Resources

- [Live Demo](https://rlhigginsjr.github.io/LAC/)
- [MCOC Legislation Page](https://www.moaamcoc.com/legislation.html)
- [HB 5280 Battle Card (PDF)](https://www.moaamcoc.com/uploads/1/4/8/4/148483887/hb_5280_battle_card_r2.pdf)
- [HB 5280 One-Pager (PDF)](https://www.moaamcoc.com/uploads/1/4/8/4/148483887/michigan_house_bill_5280_-_one-pager.pdf)
- [Michigan Legislature — Bill Search](https://www.legislature.mi.gov)
- [Google Civic Information API](https://developers.google.com/civic-information)
- [Google Cloud Console](https://console.cloud.google.com)

---

## Version History

| Version | Summary |
|---------|---------|
| v2 | State/Federal tabs; Google Civic API district lookup; three targeting modes; federal tab with coming-soon placeholder; cross-linking between related bills |
| v1 | State bills only; fixed committee targeting; mailto links; phone script |

---

## About

Developed and maintained by the Michigan Council of Chapters, MOAA.
Contributions supporting deployment benefit the Southeastern Michigan Chapter, MOAA Scholarship Fund.

© Michigan Council of Chapters, MOAA. Free to use and adapt for any MOAA Council or Chapter.
