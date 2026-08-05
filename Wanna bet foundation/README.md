# Wanna Bet Foundation

*(WannaBet prototype — self-contained build)*

A self-contained WannaBet prototype: sign-in, onboarding, and the full home app
(bets, friends, wallet, punishments, rewards, challenges). The whole thing is one
HTML file with all artwork embedded — no build step, no install, no internet
connection needed.

## Running it

**Simplest:** double-click `index.html`, or drag it into any browser.

**As a local server** (if your browser restricts `file://` for any reason):

```bash
python3 -m http.server 8000
```

Then open http://localhost:8000

**Hosting it:** drop `index.html` on any static host (Netlify, Vercel, GitHub Pages,
S3, or any web server). It's a single file with no dependencies.

## Requirements

Any modern browser (Chrome, Safari, Firefox, Edge). Designed mobile-first at 420px
max width — best viewed in a phone-sized window or device emulation.

## The flow

The app opens on the **welcome / sign-in screen**, then runs the profile-building
onboarding:

0. **Welcome** — hero artwork, a three-slide motto carousel, and the three sign-in
   buttons (see "Welcome screen" below)
   - **Continue with Apple / Google** — simulated auth: an overlay appears for a beat,
     then the flow drops into onboarding. `authProvider` records which one was used.
   - **Continue with Email** — email entry (validated) → 6-digit code screen with
     auto-advancing boxes, paste support, and a 30-second resend cooldown. It's a demo
     build, so any 6 digits verify.
   - **Terms of Service / Privacy Policy** — open a bottom sheet with placeholder copy.
1. Intro (mascot waving)
2. Name
3. Goal — single-select
4. Fun fact interstitial
5. Workout types — multi-select, max 3
6. Workout frequency — slider
7. Competitiveness — slider
8. **Odds screen** — sports-book-style moneyline odds computed live from workout
   frequency + competitiveness + workout variety
9. Interests — multi-select, max 5
10. Occupation — optional
11. Fun fact interstitial
12. Punishments you're okay with / hard off-limits — the app cross-references both
    lists into an allowed "punishment pool"
13. Apple Health permission
14. Notifications permission
15. Ready screen (mascot thumbs-up + confetti) — "Enter WannaBet" creates the account
    and drops you into the home app

Every onboarding step from *Name* onward has a back button next to the progress bar,
so answers can be changed without restarting.

## The app

Once onboarding is done the account is saved to `localStorage`, and every launch after
that opens straight on **Home** — no repeat onboarding. Signing out keeps the account
(signing back in skips onboarding); *Delete account* in Settings is the only thing that
sends you back through it.

**Home** is a launchpad, about two screens tall: header (avatar → Profile, 🔥 streak →
Streaks, wallet pill → Wallet with a **+** for Add funds, bell → Notifications) · hero
banner with *Find a Bet* · any urgent notice · **Your Overview** rail · a one-line
**odds tier** row · **Live Bets** (each card showing your odds) · **This Week** (the
weekly challenge, featured) · **Your Friends** · **Explore** — six tiles that are the
map of the app: Streaks, Predictions, Leaderboard, Insights, Recovery, and
Community (which swaps to Punishments when you owe one). Bottom tabs: Home, Bets,
➕ Create Bet, Activity, Profile.

Every tile carries its own live number — streak days, open predictions, your rank,
insight count, recovery score — so the grid doubles as a status board. The deeper
content that used to sit on Home lives in the hubs those tiles open.

Every card header, pill and CTA opens a named hub — Your Overview, Smart Odds, Streaks,
Prediction Bets, Challenges, Leaderboard, Insights, Recovery, Community, Wallet,
Notifications, Punishment centre — each with a back button. 330 handlers are wired; none
are decorative.

### Look and theme

The app follows the lime-and-black identity from the design: black pill CTAs, a lime
Create Bet button with a ring, the level ring around your avatar, and the `wannabet`
wordmark in the header (it appears from 404px up, where the design's own layout has room
for it). **There are no emoji anywhere** — every glyph is an inline SVG from the icon set
at the top of the app script (`IP` / `ic()` / `G()`), drawn to match the mockup: sneaker,
solid trophy, dollar, trend arrow, flame, dumbbell.

**Dark mode is parked** for now — the full dark palette is still in the stylesheet under
`[data-theme="dark"]`, but `applyTheme()` pins the app to light and the Appearance
control is hidden. To bring it back, restore the two-line body of `applyTheme()` and
make `appearanceCard()` return `appearanceCardOff()`.

The four Overview icons are cropped straight out of the design PNG (`MOCK_ICON`), so
they are pixel-identical to it; everything else is the SVG set.

The hero is a three-slide carousel; the dots are tappable and swipeable, it auto-advances
every six seconds, and all three slides carry the same amount of copy so the card never
resizes between them.

### Verification, Health and live sessions

- **Duration is locked.** Once a bet is created its length can't be changed, and the bet
  detail says so.
- **First to the target wins outright.** The moment anyone clears it the bet settles.
  If the clock runs out with nobody there, the highest **fit score** wins — progress
  against target, verified clips, and how consistently you showed up (`fitScore()`).
  A 30-second timer plus every commit runs `checkAutoSettle()`.
- **Apple Health is required for distance and step bets**, not optional. You can't
  create or join one without connecting, and miles sync from the provider rather than
  being typed in (Health hub → Sync now). A browser can't reach HealthKit, so this build
  implements the same contract with a stand-in provider and says so on the screen.
- **Video proof** for workout and minute bets: record in-app (real `MediaRecorder` +
  camera) or upload. Either way the clip is checked — recorded inside the bet window,
  captured in the last 12 hours, at least 8 seconds, and movement detected on the motion
  sensor while recording. An old file from the camera roll fails on its own `lastModified`
  stamp and credits nothing.
- **Live sessions**, solo or group: the camera opens, reps are counted from the
  accelerometer where the device exposes it (tap is the fallback), everyone in the room
  shows on the same board, and the first person to hit the session target wins the bet
  on the spot — the winner is assigned from the tracked count, and a `live-motion` proof
  is written to the bet. Other people's pace in the room is simulated in this prototype.

### Smart odds

Your tier (Heavy Underdog → Heavy Favorite) is read straight off your settled win rate,
so 70% puts you in *Even Match* at +100. Per-bet odds start there and move with the size
of the field and how much clock is left, which is where the `+120` / `-175` on each live
bet card comes from. Prediction multipliers use the same maths per person.

### Weekly challenges

One challenge per calendar week, generated rather than stored, so there is always a live
one. The name comes from the week itself: a real fitness event if the week has one
(Global Running Day Dash, Turkey Trot Trials, Heart Month Hustle), a seasonal title
(Summer Shred, Frostbite Fifty), otherwise a rotating set — Monday Mayhem, Iron Week,
Grit Week. Each theme carries its own gradient-and-glyph artwork, so the card looks
different week to week. Progress is the number of days that week with a logged workout.
Edit `WEEK_EVENTS`, `WEEK_NAMES` and `WEEK_ART` to change the rotation.

### Insights

Written like a coach who has read your log: your best training weekday, the format you
win most, whether small or large stakes suit you, the friend who is beating you head to
head, an overdue punishment, a streak that dies at midnight. All computed — nothing is
generated text pretending to be analysis.

### Recovery

Derived from training load: sessions in the last three days against the weekly average,
plus rest days and streak. Sleep and HRV are shown as unavailable rather than invented,
since a web prototype can't read Health.

Things that actually work: joining a bet with a custom stake, logging progress, settling
a bet (which pays out or hands you a punishment), the five-step create-a-bet flow,
wallet deposits and withdrawals with validation, invites by code or email, notification
read state, challenge join/leave, editing your name, goal, workouts and interests,
punishment rules, and notification toggles.

### Punishments and rewards

`PUNISHMENTS` and `REWARDS` are catalogs near the top of the app script. Each punishment
carries a severity (1–5) and a category that your off-limits rules can block. **Face
Reveal** is the headline one: lose a bet with it attached and it lands in the punishment
centre, where the reveal screen runs a 3-2-1 countdown — using the camera if the browser
grants it, otherwise playing out the reveal on your avatar. Nothing is uploaded, sent, or
saved; served reveals show up as blurred tiles you tap to open.

Profile → *Progress & metrics* holds the punishment metrics (served, owed, average
severity taken, face reveals completed, debts paid on time) and the reward metrics
(rewards earned, trophies, streak shields, lifetime XP).

### Where the numbers come from

Nothing on the home screen is typed into markup. Wins, losses, win rate, winnings,
wallet balance, level and XP, streaks, workouts-this-week, friend records and
head-to-head are all derived from one store: bets with participants and winners, a
wallet ledger, an XP event log and a workout log. Win a bet and the record, wallet,
XP and level all move together.

A demo history (5 friends, 10 settled bets, a workout log) is seeded on first launch so
the screens have something to add up — a banner on Home says so. Profile → *Demo data*
clears it, and the app then honestly reads 0-0, $0, level 1.

## Welcome screen

The three dots under the motto are a real carousel. Each slide is one entry in the
`MOTTOS` array near the top of the welcome section of the script:

```js
const MOTTOS = [
  { a: 'Bet on effort.',    b: 'Win together.' },
  { a: 'Miss a workout.',   b: 'Pay the tab.' },
  { a: 'Every rep counts.', b: 'Every skip costs.' },
];
```

`a` is the black line, `b` is the orange line. Add or remove entries and the dots
follow automatically. Slides advance on tap of a dot, horizontal swipe (touch) or
drag (mouse), left/right arrow keys, and on their own every 5 seconds — the timer
resets after any manual interaction and pauses while a sheet is open.

Layout is height-responsive: the artwork flexes to fill whatever room is left, and
type and button padding step down under 700px and 580px of viewport height, so the
whole screen fits without scrolling from an iPhone SE up to a desktop window.

## Editing

Everything lives in one `<script>` tag at the bottom:

- **State** — variables at the top (`userName`, `goal`, `workouts`, etc.)
- **Screens** — one `render*()` function each
- **Routing** — the `render()` switch dispatches on `step`; `nav('stepName')` moves
  between screens
- **Odds formula** — `statScore()` and `toAmericanOdds()`
- **Punishment logic** — `computePunishmentPool()`
- **Welcome carousel** — `MOTTOS`, `goMotto()`, `shiftMotto()`, `startMottoAuto()`
- **Sign-in** — `signInWith()` (swap the simulated overlay for a real SDK call here),
  `submitEmail()`, `verifyCode()`
- **Sheets** — `showOverlay()` / `closeSheet()`; legal copy lives in `LEGAL`

App section (everything after `const DB_KEY`):

- **Store** — `DB`, `saveDB()` / `loadDB()`, `blankDB()`, `seedDemo()`, `clearDemo()`
- **Derived metrics** — `betsWon()`, `winRate()`, `winningsNet()`, `balance()`,
  `levelInfo()`, `streakDays()`, `friendRecord()`, `headToHead()`, `severityIndex()`
- **Actions** — `joinBet()`, `createBet()`, `logProgress()`, `settleBet()`,
  `servePunishment()`, `withdraw()`, `deposit()`, `sendInvite()`, `joinChallenge()`;
  each one saves and re-renders through `commit()`
- **Catalogs** — `PUNISHMENTS`, `REWARDS`, `METRICS`, `XP`, `LEVEL_TITLES`
- **Routing** — `TABS` / `SCREENS` maps, `setTab()`, `go()`, `back()`; add a screen by
  writing `screenX()` and registering it in `SCREENS`
- **Odds & predictions** — `ODDS_TIERS`, `myTier()`, `betOdds()`, `personOdds()`,
  `placePrediction()`, `resolvePredictions()`
- **Weekly challenges** — `WEEK_EVENTS`, `WEEK_NAMES`, `WEEK_ART`, `weeklyChallenge()`,
  `challengeProgress()`
- **Other hubs** — `leaderboard()`, `insights()`, `recovery()`, `communityEvents()`
- **Interaction layer** — ripples on every tappable element, reveal-on-scroll, a
  condensing sticky header, drag-to-scroll rails and a light haptic tick, all in
  `setupReveal()` / `setupRails()` / `setupScrollFx()`
- **Account** — `ensureAccount()`, `finishOnboarding()`, `signOut()`, `deleteAccount()`,
  `boot()`

Colors, type, spacing, radii, shadows and easing curves are all CSS custom properties
in `:root` at the top of the `<style>` block — change the palette there rather than
hunting through markup.

## Artwork

The artwork is embedded directly in `index.html` as base64 data URIs (that's why the
file is ~1.4 MB). The originals are in `source-assets/` if you want to swap or
re-encode them:

- `welcome-hero.jpg` — welcome screen (logo + the three mascots, cropped from the
  full-screen render; the buttons, motto and legal line below it are live HTML)
- `home-banner.webp` — the dark hero banner on Home, cropped from `home-mockup.png`;
  the headline, copy and *Find a Bet* button on top of it are live HTML
- `home-mockup.png` — the original home screen design, for reference
- Profile pictures are **generated** from initials at runtime (`avatarURI()`), so no
  stock photos of real people ship in the file
- `mascot-wave.mp4` — intro screen (autoplaying loop)
- `mascot-talk.png` — name screen
- `mascot-thumbsup.png` — ready screen

To replace one, base64-encode the new file and update the matching `MASCOT_*` constant
near the top of the script:

```bash
base64 -i your-image.png | pbcopy
```

Then paste into `const MASCOT_TALK = 'data:image/png;base64,<paste here>';` — the
welcome artwork uses the same pattern under `const HERO_WELCOME`.
