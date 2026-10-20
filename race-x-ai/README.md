# Race x AI Bot

The app is `index.html`. Its Android build, Java included, is
`.github/workflows/race-x-ai.yml`, which runs on every push that changes this
folder or the workflow.

## Two repository secrets

This repo is public, so neither secret is in the code. Add both under
**Settings → Secrets and variables → Actions → New repository secret**:

| Secret | What it is | Without it |
|---|---|---|
| `RXA_SUPABASE_KEY` | the cloud-sync key | The app runs local-only, the same as it does offline. |
| `RXA_KEYSTORE_B64` | the signing keystore, base64 | The APK gets a throwaway key and **will not install over the app already on your phone**. Do not uninstall to get round it, because uninstalling deletes your logged races. |

## What the board reader does in the last seconds

- While betting is open, it re-reads the board's prices, pools and clock
  every 80ms, and every 25ms in the last 5 seconds.
- It tells the app the moment betting closes, and which prices it closed at.
- Each race is logged with that closing market (`closeOdds`, `closeBets`,
  `closeAt`), next to the last open reading (`liveOdds`).
- The Value page counts a race on the price it **closed** at. A pick that read
  1.7 at "1s" and closed at 1.6 is not counted under a 1.7 floor.
- The Price panel and the Value page price the same two vehicles. When the
  call locks with 10 seconds left, the panel and the voice switch to the
  locked pair, which is the pair the race is recorded and judged on.
- Prices are filed by vehicle name, and only on the board they were read from.
  A race whose vehicles were corrected by hand, or that was never read, gets
  no price, and Value leaves it out rather than guessing.

## Safe Bet

The home screen's 🧠 SAFE BET box says PLAY only when the Human Brain gives a
pair at least 95% and both of its vehicles pay at least 1.7x; every other race
is a SKIP. On 2,303 real races it played about one in eight, won 91.8%, never
lost more than two in a row and returned +5.5% on the closing prices. Its own
record (CTWL, last 10, return) is on the home screen and the Value page.

Both halves are checked before the APK is built:

- 55 board-reader checks run on a desktop (`desktop-check/RaceWatcherCheck.java`).
- The embedded `rxa-*-tests` scripts in `index.html` run in Node.
