# Step Alarm for Fitbit Charge 6 (iPhone)

At **5:00 AM** your Charge 6 starts buzzing, and it doesn't stop until you've
walked **25 steps**.

## How it works

The Charge 6's built-in alarm can't be tied to steps, so this setup uses your
iPhone. Like [Hourly Buzz](README.md), it relies on the Fitbit app forwarding
each iPhone notification to your wrist as one buzz.

Two shortcuts talk to each other through a small text file, `step-alarm.txt`,
in the Shortcuts folder of iCloud Drive:

| Shortcut | Started by | What it does |
|----------|------------|--------------|
| **Step Alarm** | Six automations: 5:00, 5:05, 5:10, 5:15, 5:20, 5:25 | Posts a notification every 5 seconds for about 4 minutes. Before each buzz it checks the file; if the file holds today's date, it stops. |
| **Walk It Off** | You, once you're up and holding your phone | Every 5 seconds, asks Apple Health how many steps you took in the last 10 minutes. At 25 or more, writes today's date to the file, which stops the buzzing. |

```
5:00  Step Alarm ──► buzz… buzz… buzz… (every 5 s, ~4 min)
5:05  Step Alarm ──► buzz… buzz… buzz…      ▲ each buzz first checks step-alarm.txt
5:10  …and so on until 5:25 (about 30 minutes of buzzing at most)

You get up, unlock the phone, run Walk It Off, start walking (keep the phone on you)
      Walk It Off ──► steps in the last 10 min = 8… 17… 26 ✓
                      writes today's date to step-alarm.txt ──► buzzing stops
```

### Why it's built this way

- **Six short automations instead of one long one.** iOS stops a shortcut
  that's been running in the background for more than about 4 minutes. One
  30-minute alarm loop would go quiet around 5:04. Six 4-minute loops, each
  started by its own automation, give you a full 30-minute alarm window even if
  iOS kills one of them.
- **Two shortcuts, not one.** iOS hides Health data while the phone is locked.
  A shortcut that reads steps on a locked phone fails and stops dead, which
  would kill the alarm. So *Step Alarm* only buzzes and never touches Health.
  The step check runs from *Walk It Off*, which you start after unlocking.
- **The stop signal is today's date.** *Walk It Off* writes today's date to the
  file, and *Step Alarm* stops when it sees today's date. Nothing ever has to
  reset the file: yesterday's date doesn't count, and a later 5:05 or 5:10
  instance can't accidentally re-arm the alarm.
- **Steps come from the iPhone, not the Charge 6.** Fitbit doesn't share its
  step data with Apple Health, so keep the phone on you while you walk (in
  your hand or a pocket).

If *Walk It Off* stops for any reason, the alarm keeps buzzing. Unlock the
phone and run it again.

## Setup

About 15 minutes, on iOS 17 or later. If you already set up
[Hourly Buzz](README.md), Step 1 is done.

### Step 1: Let Fitbit forward Shortcuts notifications

1. Open the **Fitbit** app → **Charge 6** → **Notifications** →
   **App Notifications**, and turn on **Shortcuts**.
2. On the iPhone, open **Settings → Notifications → Shortcuts** and make sure
   **Allow Notifications** is on.

### Step 2: Make sure nothing silences the alarm at 5 AM

This is the step people miss.

- **Charge the phone overnight.** Shortcuts runs very slowly when the screen
  is off and the phone isn't charging, which stretches the 5-second gaps
  between buzzes to much longer. On a charger it runs at full speed.
- **Charge 6 Sleep Mode or Do Not Disturb:** either one blocks notification
  buzzes. If you've scheduled Sleep Mode on the tracker (**Settings → Quiet
  modes → Sleep Mode → Schedule**), set it to end at 5:00 AM or earlier.
- **iPhone Sleep Focus:** go to **Settings → Focus → Sleep → Apps** and add
  **Shortcuts** to the allowed apps list. Otherwise iOS holds the
  notifications until your wake-up time.
- **Health access:** the first time *Walk It Off* runs, iOS asks for
  permission to read **Steps**. Allow it.

### Step 3: Create the "Step Alarm" shortcut

**Shortcuts** → **+** → name it **Step Alarm**, then add these actions:

1. **Date**: leave it as *Current Date*.
2. **Format Date**
   - Date: *Date* (from action 1)
   - Date Format: **Custom**
   - Format String: `yyyy-MM-dd`
3. **Repeat** `50` times (50 × 5 s ≈ 4 minutes). Inside the repeat block:
   1. **Get File**
      - Service: iCloud Drive, file path `step-alarm.txt` (in the Shortcuts
        folder)
      - Turn **off** *Error If Not Found*
   2. **Get Text from Input** (input: *File*)
   3. **If** *Text* **contains** *Formatted Date* (choose the variable from
      action 2, not typed text)
      - Inside the If branch: **Stop This Shortcut**
   4. **End If**. Leave the *Otherwise* branch empty.
   5. **Show Notification**: `Get up and walk 25 steps! 🚶`
   6. **Wait**: `5` seconds

When you're done, it should read:

```
Date  (Current Date)
Format Date  Date  →  Custom "yyyy-MM-dd"
Repeat 50 times
    Get File  step-alarm.txt
    Get Text from Input  File
    If  Text  contains  Formatted Date
        Stop This Shortcut
    End If
    Show Notification "Get up and walk 25 steps! 🚶"
    Wait 5 seconds
End Repeat
```

### Step 4: Create the "Walk It Off" shortcut

**Shortcuts** → **+** → name it **Walk It Off**, then add these actions:

1. **Repeat** `240` times (240 × 5 s = 20 minutes). Inside the repeat block:
   1. **Find Health Samples**
      - Type: **Steps**
      - Add filter: **Start Date** → **is in the last** → `10` **minutes**
      - Leave the limit off
   2. **Calculate Statistics**: **Sum** of *Health Samples*
   3. **If** *Statistics* **is greater than or equal to** `25`
      - Inside the If branch:
        1. **Date**: *Current Date*
        2. **Format Date**: *Date*, Custom, `yyyy-MM-dd`
        3. **Save File**
           - Input: *Formatted Date*
           - Turn **off** *Ask Where to Save*
           - Destination path: `step-alarm.txt`
           - Turn **on** *Overwrite If File Exists*
        4. **Show Notification**: `Alarm off. Good morning! ☀️`
        5. **Stop This Shortcut**
   4. **End If**
   5. **Wait**: `5` seconds

When you're done, it should read:

```
Repeat 240 times
    Find Health Samples  Steps  where Start Date is in the last 10 minutes
    Calculate Statistics  Sum of Health Samples
    If  Statistics ≥ 25
        Date  (Current Date)
        Format Date  Date  →  Custom "yyyy-MM-dd"
        Save File  Formatted Date → step-alarm.txt  (overwrite)
        Show Notification "Alarm off. Good morning! ☀️"
        Stop This Shortcut
    End If
    Wait 5 seconds
End Repeat
```

**Make it one tap:** add *Walk It Off* somewhere you can reach half-asleep.
Add a Shortcuts widget to your Lock Screen or Home Screen; on iPhone 15 Pro or
later you can also assign it to the Action Button. Another option is Back Tap:
**Settings → Accessibility → Touch → Back Tap → Double Tap → Walk It Off**.

### Step 5: Create the six alarm automations

Repeat this for **5:00, 5:05, 5:10, 5:15, 5:20 and 5:25 AM**:

1. **Shortcuts** → **Automation** → **+** → **Time of Day**.
2. Set the time and **Daily**. To skip weekends, choose **Weekly** and pick
   the days you want.
3. Choose **Run Immediately** and turn **off** *Notify When Run* (otherwise
   you get an extra buzz).
4. Pick the **Step Alarm** shortcut, then tap **Done**.

Want a shorter or longer alarm window? Each automation adds 5 minutes of
buzzing. Three automations (5:00, 5:05, 5:10) give 15 minutes; twelve give an
hour.

## Test it now (about 5 minutes)

Do this before trusting it at 5 AM. It works at any time of day.

**Part A: the buzz reaches your wrist while the phone is locked**

1. Put the Charge 6 on. Make sure it isn't in Sleep Mode or Do Not Disturb,
   and that your iPhone isn't in a Focus mode.
2. Make one extra **Time of Day** automation for **3 minutes from now**, *Run
   Immediately*, *Notify When Run* off, running **Step Alarm**.
3. Lock the phone and put it down. Ideally plug it in, since that's how it'll
   be at 5 AM.
4. **Expected:** at the set time, the Charge 6 starts buzzing roughly every
   5 seconds (slower if the phone isn't charging). Let it buzz a few times.

**Part B: walking stops it**

5. Unlock the phone, open Shortcuts and tap **Walk It Off**. The first time,
   allow access to Steps.
6. Keep the Shortcuts app on screen and the phone in your hand, and walk around
   the room. 25 steps is about 20 seconds of walking.
7. **Expected:** within about a minute of passing 25 steps, you get the
   "Alarm off" notification and the buzzing stops. The iPhone writes steps to
   Health in batches, so there's a short lag. Keep walking until it stops.

**Afterwards**

- Delete the test automation.
- To run the test again today, delete `step-alarm.txt` first (**Files** app →
  **iCloud Drive** → **Shortcuts**). Otherwise *Step Alarm* sees today's date
  and stops immediately. Tomorrow is a new date, so the real 5 AM run isn't
  affected.

**If something's off**

| Symptom | Likely cause and fix |
|---------|----------------------|
| Phone shows the notifications but the Charge 6 doesn't buzz | Shortcuts isn't on in Fitbit → Charge 6 → Notifications → App Notifications, or the tracker is in Sleep Mode or DND, or it's out of Bluetooth range. Make sure the Fitbit app is open in the background. |
| Nothing happens at all | The automation wasn't set to *Run Immediately*, or a Focus mode held the notifications. |
| Buzzes come far slower than every 5 seconds | The phone isn't charging. Plug it in. |
| *Step Alarm* stops right away | `step-alarm.txt` already holds today's date from an earlier test. Delete it. |
| *Walk It Off* never finishes | Health permission for Steps wasn't granted (Settings → Privacy & Security → Health → Shortcuts), or the phone was lying on a table instead of on you. |
| *Save File* shows an error | iCloud Drive is off for Shortcuts. Turn it on under Settings → your name → iCloud → iCloud Drive → Shortcuts. |

## Things to know

- **Expect a short delay after you reach 25 steps.** The iPhone saves steps to
  Health in small batches, so the buzzing may continue for a minute or so.
  Keep walking.
- **Keep the Shortcuts app on screen while you walk.** If the phone locks or
  you switch apps, *Walk It Off* slows down or stops, and the alarm keeps
  going (by design). Unlock and run it again.
- **The phone dings too.** That's a useful backup if you sleep through the
  wrist buzz. For wrist-only, go to **Settings → Notifications → Shortcuts** and
  turn off **Sounds**. That also silences the iPhone side of Hourly Buzz.
- **Changing the alarm time:** edit the six automations. Nothing in the
  shortcuts refers to the time.
- **Changing the step count:** edit the `25` in *Walk It Off* and the
  notification text in *Step Alarm*.
- **The Charge 6's own alarm:** not needed for this, but you can keep a Fitbit
  alarm at 5:00 AM as an extra wake-up. Dismissing it doesn't affect the step
  alarm.
- **Getting out of it is possible.** You could switch off the automations or
  write today's date to the file by hand. It's an alarm you build for
  yourself, so it can't fully prevent that.
