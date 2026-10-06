# Hourly Buzz for Fitbit Charge 6 (iPhone)

> **Also in this repo:** [Step Alarm](STEP_ALARM.md), a 5:00 AM alarm that
> keeps buzzing until you've walked 100 steps.

Every hour on the hour from **6 AM to 9 PM**, your Charge 6 buzzes once for
each hour on a 12-hour clock:

| Time  | Buzzes | | Time  | Buzzes |
|-------|--------|-|-------|--------|
| 6 AM  | 6      | | 2 PM  | 2      |
| 7 AM  | 7      | | 3 PM  | 3      |
| 8 AM  | 8      | | 4 PM  | 4      |
| 9 AM  | 9      | | 5 PM  | 5      |
| 10 AM | 10     | | 6 PM  | 6      |
| 11 AM | 11     | | 7 PM  | 7      |
| 12 PM | 12     | | 8 PM  | 8      |
| 1 PM  | 1      | | 9 PM  | 9      |

## How it works

The Charge 6 can't run custom apps. Fitbit's developer SDK only ever covered
the Versa and Sense smartwatches. What the Charge 6 *does* do is buzz once for
every phone notification the Fitbit app forwards to it.

So your iPhone does the work: one Shortcut reads the current hour and posts that
many notifications a few seconds apart. Sixteen Shortcuts automations run it at
the top of each hour. Each notification is forwarded to your wrist as one buzz.

```
iPhone clock hits 3:00 PM
  → Automation runs "Hour Buzz"
    → hour = 3
    → repeat 3×: Show Notification, wait 4 s
      → Fitbit app forwards each one → Charge 6 buzzes 3 times
```

## Setup

About 15 minutes. You need iOS 17 or later, where automations can run without
asking you first.

### Step 1: Let Fitbit forward Shortcuts notifications

1. Open the **Fitbit** app → tap your profile picture or the **Devices** icon → **Charge 6**.
2. Tap **Notifications** → **App Notifications**.
3. Turn on **Shortcuts**. If it isn't listed yet, finish Step 2, run the shortcut
   once by hand so iOS registers it as a notification source, then come back here.
4. On the iPhone, go to **Settings → Notifications → Shortcuts** and make sure
   **Allow Notifications** is on.

### Step 2: Create the "Hour Buzz" shortcut

Open **Shortcuts** → **Shortcuts** tab → **+**, name it **Hour Buzz**, and add
these actions in order (search for each name in the action search bar):

1. **Date**: leave it as *Current Date*.
2. **Format Date**
   - Date: *Date* (from step 1)
   - Date Format: **Custom**
   - Format String: `h` (a lowercase h gives the hour as 1–12 with no leading zero)
3. **Repeat**
   - Tap the number, then choose the *Formatted Date* variable from step 2 as the
     count.
   - Inside the repeat block, add:
     1. **Show Notification**: text `Hour Buzz` (any text works)
     2. **Wait**: `4` seconds
4. **End Repeat** is added for you automatically.

When you're done, the shortcut should read:

```
Date  (Current Date)
Format Date  Date  →  Custom "h"
Repeat  Formatted Date  times
    Show Notification  "Hour Buzz"
    Wait  4 seconds
End Repeat
```

Tap ▶︎ to test it. You should see one notification per hour of the current
time, and your Charge 6 should buzz the same number of times.

> **Why wait 4 seconds?** If notifications arrive too close together, the Fitbit
> app can merge them and the Charge 6 may buzz fewer times. If you're getting
> too few buzzes, raise the wait to 5–6 seconds.

### Step 3: Create the 16 hourly automations

Do this once for each hour: **6:00 AM, 7:00 AM, … 9:00 PM** (16 in all).

1. **Shortcuts** → **Automation** tab → **+** (or **New Automation**).
2. Choose **Time of Day**.
3. Set the time (for example **6:00 AM**), choose **Daily**.
4. Select **Run Immediately**.
5. Turn **off** *Notify When Run*. If you leave it on, you get one extra buzz.
6. Tap **Next** → **Hour Buzz** (under *My Shortcuts*) → **Done**.

**Tip:** after the first one, long-press it in the Automation list to see
whether your iOS version offers **Duplicate**. If it does, duplicate it and
just change the time each time.

## Checklist and troubleshooting

- **No buzzes at all:** check Step 1. The Charge 6 must be in Bluetooth range,
  and the Fitbit app should be running in the background (don't swipe it away).
- **Charge 6 is silent at certain times:** the Charge 6 blocks notifications
  in **Do Not Disturb** and **Sleep Mode** (swipe down on the tracker to check).
  A Focus mode on the iPhone does the same unless Shortcuts is on that Focus's
  allowed apps list.
- **Too few buzzes:** increase the **Wait** in the shortcut (see the note
  above).
- **An extra buzz each hour:** turn off *Notify When Run* on each automation.
- **Pausing it:** open any automation and toggle it off, or turn off
  **Shortcuts** under Fitbit → Charge 6 → Notifications → App Notifications to
  mute all of them at once.
- **Changing the hours:** add or delete automations. The shortcut itself works
  at any hour.
