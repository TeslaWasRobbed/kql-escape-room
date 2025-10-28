
KQL Escape Room (Self‑Serve, Shared Leaderboard)
A lightweight, static web app that runs a KQL escape room workshop.
Analysts write queries in their own Microsoft Sentinel workspace, then enter answers in the web page. A shared leaderboard is powered by Firebase Realtime Database so teams can see how they compare—no servers to run, perfect for GitHub Pages.

✨ Features

Self‑serve: single index.html, host anywhere (GitHub Pages recommended).
Analyst‑led: participants write KQL in Sentinel Logs, not in the page.
Shared leaderboard: Firebase Realtime Database + Anonymous auth.
Local fallback: still works without Firebase; stores results locally.
Facilitator controls: reveal answer keys with a passcode.
Timer & scoring: per‑team timer and automatic scoring.
Watchlist guidance: clear aliases to use in queries.
No secrets: Firebase client config is public by design; access is enforced by database rules and Anonymous auth.


🧭 Architecture
Browser (GitHub Pages)
  ├── index.html (UI, challenge flow, scoring)
  ├── Firebase Web SDK (cdn)
  └── Realtime Database (shared leaderboard; auth: anonymous)
Microsoft Sentinel (your tenant)
  └── Analysts run KQL (nothing posts back from Sentinel to this page)

No back‑end server; the only data leaving the page is leaderboard entries.

📋 Prerequisites

A Microsoft Sentinel workspace for analysts to query
GitHub Pages or any static hosting
A Firebase project (free tier is fine)

Authentication → Anonymous sign‑in enabled
Realtime Database created (we’ll add rules below)




🚀 Quick Start


Clone this repo (or add the provided index.html).


Create Firebase project (one time):

console.firebase.google.com → Add project
Authentication → Sign-in method → Anonymous → Enable
Realtime Database → Create database (pick region)
Project settings → General → Web app (</>) → Register app and copy the config JSON.



Open index.html and paste your config:
Interactivity on code previews is coming soon<!-- inside index.html --><script>  // ...  var firebaseConfig = {    apiKey: "YOUR_API_KEY",    authDomain: "YOUR_PROJECT.firebaseapp.com",    databaseURL: "https://YOUR_PROJECT-default-rtdb.REGION.firebasedatabase.app",    projectId: "YOUR_PROJECT_ID",    storageBucket: "YOUR_PROJECT.appspot.com",    messagingSenderId: "YOUR_SENDER_ID",    appId: "YOUR_APP_ID"  };  // ...</script>Show more lines


Set database security rules in Firebase → Build → Realtime Database → Rules:
JSON{  "rules": {    "leaderboards": {      "$eventId": {        ".read": "auth != null",        "scores": {          "$scoreId": {            ".write": "auth != null && newData.hasChildren(['team','score','time','when']) && newData.child('team').isString() && newData.child('score').isNumber() && newData.child('score').val() >= 0 && newData.child('score').val() <= 5 && newData.child('time').isString() && newData.child('when').isNumber()",            ".validate": "newData.child('team').val().length > 0 && newData.child('team').val().length <= 40"          }        }      }    }  }}Show more lines


Publish to GitHub Pages:

Push your repo.
Repo Settings → Pages → Branch: main (or docs/) → Save.
Share the Pages URL with teams.




✅ If Firebase is not configured or blocked, the page still works with a local leaderboard (per browser/device).


🧪 Using the App (analysts)

Open the workshop URL (GitHub Pages).
In a separate tab, open Sentinel → Logs in your training workspace.
For each stage, write and run KQL in Sentinel, then paste the answer in the page and click Check.
When all stages are correct, your score & time are automatically posted to the shared leaderboard.


📚 “How to play” — Watchlists (aliases)
If you imported the CSVs as watchlists, analysts can use:

 _GetWatchlist('SigninLogs_WL') — Sign‑in events
 _GetWatchlist('SecurityEvent_WL') — Windows logons
 _GetWatchlist('FileAccess_WL') — File access logs
 _GetWatchlist('Network_WL') — Network events


Tip: Watchlists return strings. Cast columns when needed:
todatetime(TimeGenerated), toint(...), tolong(BytesSent), etc.


🧑‍🏫 Facilitator Controls


Reveal answers: type showanswers in the facilitator box and hit Enter.
(Only affects the current browser; refresh to hide.)


Timer: click Start timer per team at the beginning.


Event scope: scores are stored under an Event ID path in Firebase.
By default, the code sets EVENT_ID to today’s date (e.g., kql-escape-2025-10-14).
You can change it in index.html:
JavaScript// e.g., per cohort or regionvar EVENT_ID = "kql-escape-EMEA-2025-10-14";Show more lines
After changing, re‑deploy so all teams post to the same event.


Reset scoreboard: delete the node /leaderboards/<EVENT_ID> in Firebase.



⚙️ Configuration & Customization
Change stage prompts / answers / hints
Look for the ANSWERS map and stage sections in index.html.


Answers (exact match / contains / regex):
JavaScriptvar ANSWERS = {  1: { type:"equals",   list:["alex.james@contoso.com"], ci:true },  2: { type:"equals",   list:["198.51.100.23"],          ci:true },  3: { type:"equals",   list:["FS-SRV01"],               ci:true },  4: { type:"contains", list:["confidential.xlsx"],      ci:true },  5: { type:"equals",   list:["203.0.113.200"],          ci:true }};Show more lines


Prompts & hints: edit the corresponding HTML in each stage block.


Facilitator passcode
JavaScriptvar PASSCODE = "showanswers";Show more lines
Theme & branding
Update header text, colors in the :root CSS block, or add your logo in the header.

🧰 Local Development

Open index.html directly in a browser, or use a static server (e.g., VS Code Live Server).
Firebase calls require network access; if offline, the page will fall back to local leaderboard only.


🔐 Security & Privacy

Auth: only Anonymous sign‑in is used.
Data written: team, score, time, when (epoch), under leaderboards/<EVENT_ID>/scores/*.
No PII beyond the team name (you can instruct teams to avoid real names).
Rules: the provided rules restrict writes to valid shapes & ranges and require auth.
Client config: Firebase web config is public by design; don’t treat it as a secret.
Local storage: the page stores your team name, answers, and local scores in localStorage (per device/profile).


🛠️ Troubleshooting


Leaderboard not loading

Check the browser console for Firebase init errors.
Ensure Anonymous Auth is enabled and Realtime Database URL matches your region path.
Verify database rules were applied and published.
Confirm EVENT_ID is the same across all teams (same deployed file).



Buttons not working

If embedded in another system (e.g., SharePoint/Confluence) with strict CSP, serve it as a standalone page or remove inline handlers and attach events via external JS with proper CSP headers.



Teams can’t see each other’s scores

Make sure everyone is using the same URL (deployed commit) and the same EVENT_ID.
Check that the Rules allow reads (auth != null) and that Anonymous Auth is actually signing in (see console).



Resetting the leaderboard

Delete /leaderboards/<EVENT_ID> in Firebase (Data view).
Or switch EVENT_ID in the file for a fresh board.




❓FAQ
Q: Can we run this without Firebase?

A: Yes. The page still works with a local leaderboard stored in the browser. Only shared standings require Firebase.

Q: Can we run multiple rooms simultaneously?

A: Yes. Change EVENT_ID (e.g., kql-escape-roomA) and deploy separate page URLs (or branches).

Q: Can we force team PINs or restrict team name changes?

A: You can add a pin field in submissions and a constant EVENT_PIN in the page. Update rules to require pin == 'XYZ'.

Q: Can you add support for ?event= in the URL?

A: Yes. Parse new URLSearchParams(location.search).get('event') and fall back to the default. (Happy to add this if you want it.)

🧑‍💻 Directory
Single‑file app:
.
├── index.html   # full app (UI, stages, scoring, Firebase integration)
└── README.md    # this file


📝 License
MIT (or your org’s standard). Add a LICENSE file if needed.

🙌 Credits

Built for hands‑on KQL practice in Microsoft Sentinel.
Shared leaderboard powered by Firebase (Anonymous Auth + Realtime DB).
