# Stupefy Setup Guide

## The System

Your website (stupefy.com) is updated from an Apple Note on your phone.

```
Apple Note → Shortcut (one tap) → GitHub → Vercel → stupefy.com
```

---

## 1. Your Apple Note Format

Create a note called **"Stupefy"** and write it exactly like this:

```
## now
The Age of Spiritual Machines Revisited | Wired
https://wired.com/2026/01/spiritual-machines

## next
Why the Brain Is So Noisy | Nautilus
https://nautilus.org/brain-noisy

The Consultancy Trap | The Generalist
https://thegeneralist.com/consultancy-trap

Brains, Bodies, and the Biology of Trauma | Aeon
https://aeon.co/essays/biology-of-trauma

## read
The K-Shaped Recovery No One Talks About | Bloomberg
https://bloomberg.com/k-shaped

Pat Metheny on Practice, Process, and Sound | JazzTimes
https://jazztimes.com/metheny-practice
```

### Rules
- `## now` = what you're currently reading (one article)
- `## next` = your queue (as many as you want)
- `## read` = finished (as many as you want)
- Each article is TWO lines: `Title | Source` then the URL on the next line
- Blank line between articles
- That's it

---

## 2. GitHub Repo Setup

1. Go to github.com → New Repository
2. Name it `stupefy` (or whatever you want)
3. Can be private
4. Add the two files from this package: `index.html` and `reading.json`
5. Create a **Personal Access Token**:
   - Go to github.com → Settings → Developer Settings → Personal Access Tokens → Fine-grained tokens
   - Name: `stupefy-shortcut`
   - Expiration: 1 year (or no expiration)
   - Repository access: Only select → pick your `stupefy` repo
   - Permissions: Contents → Read and Write
   - Generate → **copy the token and save it somewhere safe**

---

## 3. Build the Apple Shortcut

Open the **Shortcuts** app on your iPhone. Create a new shortcut called **"Update Stupefy"**.

Add these actions in order:

### Action 1: Find Notes
- **Find Notes where**
  - Name **is** `Stupefy`
- Limit: 1

### Action 2: Get text from note
- **Get Text from** `Notes`
- This extracts the plain text content

### Action 3: Run JavaScript on text (via "Run JavaScript on Webpage" or use a Text action approach)

Since Shortcuts doesn't have a native JSON parser that handles this well, we'll use a different approach:

### SIMPLER APPROACH — Use "Run Script Over SSH" or a Shortcut-native flow:

Here's the most reliable method using only native Shortcut actions:

---

### Step-by-step Shortcut actions:

**Action 1: Find Notes**
- Find Notes where Name is "Stupefy"
- Limit: 1

**Action 2: Get Body of Note**
- Get `Body` from `Notes` (the variable from Action 1)

**Action 3: Set Variable**
- Set variable `NoteText` to the output

**Action 4: Get Contents of URL (GitHub API — get current file SHA)**
- URL: `https://api.github.com/repos/YOURUSERNAME/stupefy/contents/reading.json`
- Method: GET
- Headers:
  - `Authorization`: `Bearer YOUR_GITHUB_TOKEN`
  - `Accept`: `application/vnd.github.v3+json`

**Action 5: Get Dictionary Value**
- Get `sha` from the output of Action 4
- Set variable `FileSHA`

**Action 6: Text action — build the JSON**

This is the tricky part. You need to convert the note text into JSON. The most reliable way in Shortcuts:

Use **Replace Text** actions to parse the note:

1. **Split Text** — split `NoteText` by `## ` to get sections
2. For each section, use **Match Text** (regex) or **Replace Text** to extract articles

**RECOMMENDED: Use a Shortcut that sends the note text to a small serverless function that parses it and returns JSON.**

OR

**SIMPLEST OPTION: Skip the parsing entirely. Push the raw note text to GitHub as `reading.txt`, and have the website parse it client-side with JavaScript.**

---

## 3 (REVISED): The Simplest Pipeline

Instead of parsing in the Shortcut, push the raw note as a text file. The website parses it.

### Shortcut actions (just 6 steps):

**1. Find Notes** where Name is "Stupefy" (Limit: 1)

**2. Get Body** of the note

**3. Get Contents of URL** (get current file SHA)
- URL: `https://api.github.com/repos/YOURUSERNAME/stupefy/contents/reading.txt`
- Method: GET
- Headers:
  - `Authorization`: `Bearer ghp_YOURTOKEN`
  - `Accept`: `application/vnd.github.v3+json`

**4. Get Dictionary Value** — get `sha` from result

**5. Base64 Encode** the note body text

**6. Get Contents of URL** (push update)
- URL: `https://api.github.com/repos/YOURUSERNAME/stupefy/contents/reading.txt`
- Method: PUT
- Headers:
  - `Authorization`: `Bearer ghp_YOURTOKEN`
  - `Accept`: `application/vnd.github.v3+json`
- Request Body (JSON):
```json
{
  "message": "Update reading list",
  "content": "[BASE64 ENCODED TEXT FROM STEP 5]",
  "sha": "[SHA FROM STEP 4]"
}
```

That's the entire Shortcut. 6 actions. The website JavaScript handles parsing the note format.

---

## 4. Vercel Setup

1. Go to vercel.com → Sign up with GitHub
2. Import your `stupefy` repo
3. Framework: **Other** (it's just static files)
4. Deploy — you'll get a URL like `stupefy.vercel.app`
5. Test it

---

## 5. Point stupefy.com to Vercel

1. In Vercel: Settings → Domains → Add `stupefy.com`
2. Vercel will give you DNS records (usually an A record and/or CNAME)
3. Log into Epik → DNS settings for stupefy.com
4. Remove any existing A/CNAME records pointing to WP Engine
5. Add the records Vercel gave you:
   - Type: **A** → `76.76.21.21` (Vercel's IP, they'll confirm)
   - OR Type: **CNAME** → `cname.vercel-dns.com`
6. If you want `www.stupefy.com` too, add a CNAME for `www` → `cname.vercel-dns.com`
7. Wait for DNS propagation (usually 5-30 minutes)

---

## Daily Workflow

1. Open your "Stupefy" note on your phone
2. Paste a new link, type the title, move things between sections
3. Tap the "Update Stupefy" shortcut
4. Done — site is live in ~10 seconds

