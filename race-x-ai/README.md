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

## Same Road and Diff Road

Two models that look up the board instead of reasoning about it: what happened
the last times these same three vehicles met, on the same road (Same Road) or
on any road (Diff Road). They count only the races logged on the phone -
nothing built into the app - so a reset starts them again from nothing.

- No call before ten past races of the board.
- Each vehicle is the first pick as often as it has won, and a vehicle that has
  won one time in N is never left N calls without a turn. The second pick is
  the stronger of the other two.
- They do not vote in Consensus, and Best, Adaptive and Road Expert never
  follow them.

## Safe Bet

The home screen's 🧠 SAFE BET box says PLAY only when the Human Brain gives a
pair at least 95% (98% straight after two losses in a row) and both of its
vehicles pay at least 1.7x; every other race is a SKIP. On 2,303 real races it played about one in eight, won 91.8%, never
lost more than two in a row and returned +5.5% on the closing prices. Its own
record (CTWL, last 10, return) is on the home screen and the Value page.

The cog on the Safe Bet box opens its settings: how sure the Brain must be
(90-98%) and how much a win must pay (any, or 1.10x-1.40x on a split stake).
Every combination is replayed over the races logged on the phone, and the sheet
shows which kept 90% with no more than two losses in a row. Bigger wins lose
more often; the sheet shows by how much on those races.

Under every PLAY the box gives the break-even: how often the bet has to win at
those prices to be worth making. And Safe Bet pauses by itself when the Human
Brain's last 50 races fall more than three standard errors under its usual
rate - a fuse for the day the game itself is changed.

The Brain's percentage reads high - pairs it calls 95% have won about 87 times
in 100, 98% about 92, and its "100%" about 94 - so under every PLAY the box also says what that
percentage has really won, and whether the bet has an edge at the prices on the
board. The curve is `sigmoid(a + b*logit(stated))`, fitted to the phone's own
latest races once it has 300 with the Brain's chances and to 3,172 backup races
(a 0.25, b 0.56) until then. The rule is unchanged but for one thing: a pair
that would return less than the stake even by winning (1.8 and 2.2 come back
as 0.99x on a split stake) is never played.

### What the returns can and cannot be

Measured on the 1,902 backup races that carry prices, betting the Brain's
chances every way there is:

| Rule | Plays | Won | Longest losing run | Profit on the amount staked |
| --- | --- | --- | --- | --- |
| Safe Bet as shipped (95%, any win) | 217 | 90.8% | 1 | +5.1% |
| 95%, a win must return 1.15x | 105 | 89.5% | 1 | +9.0% |
| 96%, a win must return 1.15x | 68 | 91.2% | 1 | +9.8% |
| Any pair returning 1.5x, Brain 80%+ | 75 | 73.3% | 3 | +16.5% |
| Any pair returning 1.7x, Brain 60%+ | 97 | 63.9% | 3 | +16.5% |
| One vehicle paying 1.7x+, Brain 80%+ | 93 | 63.4% | 4 | +16.7% |

A profit of 70% on the total stake needs a pair that wins 9 times in 10 and
returns 1.89x. The pairs that return 1.7x or more won 41% of the time. The
payout is the other players' money, so a pair pays that much only because most
of the money says it loses. About +10% with nine wins in ten, or about +16%
with two wins in three and longer losing runs, is what this history holds.

### Storage

The whole history is one string in the phone's storage, which holds about 5.2
million characters. A 220-race backup measured 4.14 million (18.8 KB a race):
the Road Evidence Lab kept four full hypotheses per race (10 KB) that only its
export reads, a field nothing reads held a second copy of the frozen bundle
(1.5 KB), and frozen numbers were written to 18 places. A logged race now
keeps each frame's time and leading pair, drops the copy and writes five
places: the same backup loads as 1.55 million characters, about 7 KB a race.
If the storage still fills, the oldest races give up their Road Lab evidence
(never their result, picks, prices or roads), the save is retried and the app
says so; only when nothing is left to drop does it ask for a backup and reset.

### Machine-learning test

Six backups (3,172 real races) were put through XGBoost, LightGBM and
scikit-learn (random forest, extra trees, gradient boosting, logistic
regression, a small neural net, rankers, a stack), 72 features known before
the race, every race predicted by models trained only on the races before it.

| On 2,172 unseen races | Winning pair | Longest losing run |
| --- | --- | --- |
| Human Brain (in the app) | 82.6% | 4 |
| LightGBM | 82.3% | 3 |
| XGBoost | 82.1% | 3 |
| XGBoost, settings tuned on earlier races | 82.0% | 3 |
| Stack of five models | 81.7% | 4 |
| XGBoost trained on the latest 300 races only | 80.4% | 5 |
| XGBoost with the crowd's money added (1,669 races) | 80.2% vs Brain 83.0% | 4 |

None beat the Brain, and none sorted good Safe Bets from bad any better (0.757
against 0.757-0.760 on the usual 0.5-1 scale). The reason is in the game: with
the two hidden roads known the right pair is 94.6% certain, with the track
layout too 97%, and nothing visible before the race predicts those roads - the
models guessed the exact hidden pair 14% of the time against the Brain's 10%,
and it won them no more races. So no ML model was added. What the test did
show is that the Brain's percentages are too high, which is the line above.

Both halves are checked before the APK is built:

- 55 board-reader checks run on a desktop (`desktop-check/RaceWatcherCheck.java`).
- The embedded `rxa-*-tests` scripts in `index.html` run in Node.
