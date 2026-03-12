# SatvicScan — Claude Code Brief

## Repo
https://github.com/Naitikparekh/SatvicScan

## What this app does
SatvicScan is a food compliance checker for people following the **Swaminarayan religious Satvik diet**.
The user scans a product (barcode, photo of ingredients label, or types ingredients) and Claude AI instantly tells them if it is **Safe, Not Permitted, or Caution**.

---

## Tasks for Claude Code

### 1. Set up folder structure
```
SatvicScan/
├── web/
│   ├── index.html          ← main web app (see current code below)
│   └── launch.sh           ← Mac launcher script
├── ios/                    ← (empty for now, Expo/React Native app goes here later)
├── assets/
│   └── icon.png            ← app icon placeholder
└── README.md
```

### 2. Rebuild the web UI — minimalist redesign
The current HTML file works but looks chunky. Redesign it with:

**Design principles:**
- Clean white/off-white or very dark background — no busy gradients
- Single accent color: `#4CAF50` (green) for safe, `#F44336` (red) for not safe, `#FF9800` (amber) for caution
- Large, readable typography — SF Pro or system font stack
- Generous whitespace — breathe between elements
- Smooth transitions (0.2s ease) on all interactive elements
- Mobile-first layout — max-width 390px centered (iPhone 15 Pro width)
- Rounded corners everywhere (16px radius minimum)
- Subtle shadows instead of heavy borders
- No clutter — only show what the user needs right now

**What to REMOVE from current UI:**
- The "prohibited ingredients" strip listing all the banned items — DELETE IT ENTIRELY
- The "Zero Tolerance" rules box — DELETE IT
- Any visual indication of what diet rules are being applied — the user already chose Swaminarayan, they don't need reminders

**What to KEEP:**
- Scan options: Photo (camera/file), Barcode (type number), Manual (paste text)
- Result card: verdict (SAFE / NOT PERMITTED / CAUTION), summary, ingredient flags, analysis
- Recent scans history list
- API key entry (first launch only — stored in localStorage, never asked again)
- Mac webcam support via getUserMedia

**New home screen layout:**
```
[App icon + "SatvicScan" wordmark]

[Large subtitle: "Is this food Satvik-safe?"]

[3 scan option cards in a clean grid:]
  📷 Scan Label     📊 Barcode
  ✏️  Type / Paste  (full width)

[Recent scans — minimal list, just product name + verdict dot]
```

**Result card redesign:**
- Big verdict at top: ✅ SAFE / ❌ NOT PERMITTED / ⚠️ CAUTION
- One-line summary below verdict
- Expandable "Details" section (collapsed by default) showing ingredient flags
- No Category A/B/C labels shown to user — just flag the bad ingredient and say why simply
- Example: "Contains onion powder — not permitted in Swaminarayan diet"

### 3. API key persistence
- Store in `localStorage` under key `satvicscan_apikey`
- Load on app start — if exists, skip the API key modal entirely
- Show a small settings icon (⚙) in top right corner that lets user update/clear key
- API key modal: minimal — just a password input field and Save button
- Validation: must start with `sk-ant` — show inline error if wrong format

### 4. Diet rules — hardcoded, invisible to user
The Swaminarayan Satvik diet rules are already embedded in the AI system prompt.
The user sees **one preset: "Swaminarayan Diet"** — it is always selected, no toggle needed.
Do NOT show any list of what is excluded. Just run the checks silently.

**The system prompt to use for Claude API calls:**
```
You are a Satvik Swaminarayan diet compliance checker. Strict decision algorithm.

CORE PRINCIPLE: ANY trace of a prohibited ingredient = REJECT. Unknown source = REJECT. Default = NOT_SAFE.

CATEGORY A — DIRECTLY PROHIBITED (zero tolerance):
Animals: all meats, poultry, fish, seafood, eggs (any form), lard/tallow/suet, animal broth/stock, gelatin, animal rennet, isinglass, L-Cysteine/E920, Carmine/E120, Shellac/E904, Lysozyme/E1105, collagen, E631, E635, bone char.
Alliums (ALL forms — fresh/dried/powder/extract/oil): Onion, Garlic, Spring onion, Green onion, Leek, Chives, Shallots, Scallions, Wild garlic, Ramps, Asafoetida/Hing.
Alcohol: beer, wine, spirits, kombucha, vanilla extract (alcohol-based), malt vinegar, wine vinegar.

CATEGORY B — AMBIGUOUS (unknown source = REJECT):
Glycerin/Glycerol (need "vegetable"), Enzymes (need "microbial"), Mono & Diglycerides/E471 (need "plant"), Natural flavors/flavouring (REJECT — may hide onion/garlic/meat/alcohol), Spices unspecified (REJECT), Seasoning unspecified (REJECT), Lecithin/E322 (need "soy"), Omega-3 (need "algae"), Vitamin D3 (need "plant-based"), Stearic acid (need "plant").

CATEGORY C — CROSS-CONTAMINATION: "may contain" or "shared equipment" with prohibited = CAUTION.

Respond ONLY with valid JSON, no markdown:
{
  "verdict": "SAFE" | "NOT_SAFE" | "CAUTION",
  "productName": "string",
  "summary": "one plain-English sentence",
  "flags": [
    {
      "type": "bad" | "caution" | "good",
      "ingredient": "exact name from label",
      "reason": "plain English reason — e.g. contains onion powder, not permitted"
    }
  ],
  "ambiguous": ["list of ingredients needing manufacturer verification"],
  "analysis": "short paragraph",
  "confidence": "HIGH" | "MEDIUM" | "LOW"
}
```

### 5. API calls
- Model: `claude-haiku-4-5-20251001` (cheapest, fast, accurate enough)
- Headers required:
  ```
  Content-Type: application/json
  x-api-key: <user's key>
  anthropic-version: 2023-06-01
  anthropic-dangerous-direct-browser-access: true
  ```
- Endpoint: `https://api.anthropic.com/v1/messages`
- For photo scans: compress image to max 1024px before sending, JPEG quality 0.88
- For barcode: first try Open Food Facts `https://world.openfoodfacts.org/api/v2/product/{code}.json`, then pass ingredients to Claude. If product not found, Claude gives CAUTION and tells user to scan ingredients label.

### 6. Mac launcher (web/launch.sh)
```bash
#!/bin/bash
DIR="$(cd "$(dirname "$0")" && pwd)"
PORT=8080
lsof -ti:$PORT | xargs kill -9 2>/dev/null
sleep 0.5
cd "$DIR"
python3 -m http.server $PORT &>/dev/null &
sleep 1
open -a "Google Chrome" "http://localhost:$PORT/index.html" 2>/dev/null || open "http://localhost:$PORT/index.html"
echo "✅ SatvicScan running at http://localhost:$PORT"
wait
```

### 7. README.md
Write a clean README with:
- What the app is
- How to run locally (the launcher script)
- How to get a Claude API key
- Folder structure explanation
- Note that iOS app coming soon

### 8. Git commit and push
Once all files are in place:
```bash
git add .
git commit -m "feat: minimalist UI redesign, persistent API key, clean folder structure"
git push origin main
```

---

## Current working files location
The working HTML file and launcher are at:
- `/mnt/user-data/outputs/satvik-scanner-final.html` — use this as the base for the redesign
- `/mnt/user-data/outputs/launch-satvikscan.sh` — use this as base for `web/launch.sh`

Read these files first, then rebuild into the new folder structure.

---

## iOS app (next phase — do not build yet)
After web is done, the iOS app will be built with:
- **Expo + React Native** (so it works on iOS and Android)
- Same Claude API integration
- API key stored in **Expo SecureStore** (encrypted, iOS Keychain backed)
- Native camera access via `expo-camera`
- Native barcode scanning via `expo-barcode-scanner`
- Same diet logic, same AI prompt

---

## Important notes
- App name spelling: **SatvicScan** (with a C, not K) — match the GitHub repo name
- No onboarding screens, no sign-up, no accounts — just open and scan
- Keep it fast — the goal is: open app → scan → result in under 10 seconds
- The user is a BAPS Swaminarayan follower — the diet rules are religious, treat them with full strictness
