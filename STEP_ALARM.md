# Step Alarm for Fitbit Charge 6 (iPhone)

At **5:00 AM** your Charge 6 starts buzzing, and it doesn't stop until you've
walked **100 steps**.

## How it works

The Charge 6's built-in alarm can't be tied to steps, so this setup uses your
iPhone. Like [Hourly Buzz](README.md), it relies on the Fitbit app forwarding
each iPhone notification to your wrist as one buzz.

Two shortcuts share a small text file, `step-alarm.txt`, in iCloud Drive:

| Shortcut | Started by | What it does |
|----------|------------|--------------|
| **Step Alarm** | An automation at 5:00 AM | Writes `ringing` to the file, then posts a notification every 5 seconds until the file says `done`. Gives up after 30 minutes as a safety stop. |
| **Walk It Off** | You, once you're up and holding your phone | Checks Apple Health for steps taken since 5:00 AM every 5 seconds. At 100 or more, writes `done` to the file, which stops the buzzing. |

```
5:00 AM  Step Alarm ──► buzz… buzz… buzz… (every 5 s)
             ▲ checks step-alarm.txt each time
You get up, unlock the phone, run Walk It Off, start walking
         Walk It Off ──► Health steps since 5:00 = 37… 82… 104 ✓
                         writes "done" ──► Step Alarm stops
```

### Why it takes two shortcuts

- **iOS hides Health data while the phone is locked.** A shortcut that reads
  step counts at 5:00 AM on a locked phone fails and stops, which would kill
  the alarm. So the 5:00 AM shortcut only buzzes and never touches Health. The
  step check runs from a second shortcut that you start after unlocking.
- **Your steps come from the iPhone, not the Charge 6.** Fitbit doesn't share
  its step data with Apple Health, so carry your phone while you walk (a
  pocket or your hand is fine).

If *Walk It Off* stops for any reason (for example, the phone auto-locks), the
alarm keeps buzzing. Just unlock the phone and run *Walk It Off* again. Steps
you've already taken since 5:00 AM still count.

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

1. **Text**: `ringing`
2. **Save File**
   - Input: *Text*
   - Turn **off** *Ask Where to Save*
   - Destination path: `step-alarm.txt` (saves into the Shortcuts folder in
     iCloud Drive)
   - Turn **on** *Overwrite If File Exists*
3. **Repeat** `360` times (360 × 5 s = 30 minutes). Inside the repeat block:
   1. **Get File** from the Shortcuts folder
      - Path: `step-alarm.txt`
      - Turn **off** *Error If Not Found*
   2. **If** *File* **contains** `done`
      - Inside the If branch: **Stop This Shortcut**
   3. **End If**. Leave the *Otherwise* branch empty, or delete it.
   4. **Show Notification**: `Get up and walk 100 steps! 🚶`
   5. **Wait**: `5` seconds

When you're done, it should read:

```
Text "ringing"
Save File  Text → step-alarm.txt  (overwrite)
Repeat 360 times
    Get File  step-alarm.txt
    If  File  contains "done"
        Stop This Shortcut
    End If
    Show Notification "Get up and walk 100 steps! 🚶"
    Wait 5 seconds
End Repeat
```

### Step 4: Create the "Walk It Off" shortcut

**Shortcuts** → **+** → name it **Walk It Off**, then add these actions:

1. **Date**: type `5:00 AM`. This gives today at 5:00 AM.
2. **Repeat** `240` times (240 × 5 s = 20 minutes). Inside the repeat block:
   1. **Find Health Samples**
      - Type: **Steps**
      - Add filter: **Start Date** *is after* → choose the *Date* variable from
        action 1
      - Leave the limit off
   2. **Calculate Statistics**: **Sum** of *Health Samples*
   3. **If** *Statistics* **is greater than or equal to** `100`
      - Inside the If branch:
        1. **Text**: `done`
        2. **Save File**: *Text* → `step-alarm.txt`, *Ask Where to Save* off,
           *Overwrite If File Exists* on
        3. **Show Notification**: `Alarm off. Good morning! ☀️`
        4. **Stop This Shortcut**
   4. **End If**
   5. **Wait**: `5` seconds

When you're done, it should read:

```
Date "5:00 AM"
Repeat 240 times
    Find Health Samples  Steps  where Start Date is after Date
    Calculate Statistics  Sum of Health Samples
    If  Statistics ≥ 100
        Text "done"
        Save File  Text → step-alarm.txt  (overwrite)
        Show Notification "Alarm off. Good morning! ☀️"
        Stop This Shortcut
    End If
    Wait 5 seconds
End Repeat
```

**Make it one tap:** add *Walk It Off* somewhere you can reach half-asleep.
For example, add a Shortcuts widget to your Lock Screen or Home Screen. On
iPhone 15 Pro or later you can also assign it to the Action Button. Another
option is Back Tap: **Settings → Accessibility → Touch → Back Tap →
Double Tap → Walk It Off**.

### Step 5: Create the 5:00 AM automation

1. **Shortcuts** → **Automation** → **+** → **Time of Day**.
2. Set **5:00 AM** and **Daily**. To skip weekends, choose **Weekly** and pick
   the days you want.
3. Choose **Run Immediately** and turn **off** *Notify When Run*.
4. Choose the **Step Alarm** shortcut, then tap **Done**.

### Step 6: Test it before relying on it

1. Temporarily change the automation to two minutes from now, then lock your
   phone and put it down.
2. Check that the Charge 6 starts buzzing on time **while the phone is
   locked**.
3. Unlock the phone, run **Walk It Off** and walk around. The buzzing should
   stop shortly after you pass 100 steps.
4. Change the automation back to 5:00 AM.

## Things to know

- **Expect a short delay after you reach 100 steps.** The iPhone writes steps
  to Health in small batches, so the buzzing may continue for up to a minute
  or so. Keep walking.
- **Stop the iPhone from locking mid-walk.** If *Walk It Off* stops, the alarm
  keeps going (by design). Keep the screen on while you walk, or set **Settings
  → Display & Brightness → Auto-Lock** to a few minutes.
- **The phone dings too.** That's a useful backup if you sleep through the
  wrist buzz. For wrist-only, go to **Settings → Notifications → Shortcuts** and
  turn off **Sounds**. That also silences the iPhone side of Hourly Buzz.
- **The safety stop is 30 minutes.** To change it, adjust *Step Alarm*'s
  Repeat count (12 = 1 minute). To require more or fewer steps, change the `100`
  in *Walk It Off*.
- **Changing the alarm time:** change the time in both the automation (Step 5)
  and the **Date** action in *Walk It Off* (Step 4, action 1).
- **The Charge 6's own alarm:** you don't need it for this, but you can keep a
  Fitbit alarm at 5:00 AM as an extra wake-up. Dismissing it doesn't affect
  the step alarm.
- **Getting out of it is possible.** You could switch off the automation or
  edit the file. It's an alarm you build for yourself, so it can't fully
  prevent that.
