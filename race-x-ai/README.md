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

Both halves are checked before the APK is built:

- 38 board-reader checks run on a desktop (`desktop-check/RaceWatcherCheck.java`).
- The embedded `rxa-*-tests` scripts in `index.html` run in Node.
