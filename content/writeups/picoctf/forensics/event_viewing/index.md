+++
title = "Event Viewing"
date = '2026-01-30T02:59:54-05:00'
+++

**Goal:** Investigate a suspicious Windows system using Event Viewer logs and recover a flag split across multiple events.

---

## Challenge Description

Screenshot from the picoCTF challenge page:

![Screenshot of the picoCTF Event-Viewing challenge description](challenge.png)

---

## Challenge Overview

This challenge simulates a real-world incident response scenario: a user installs some “Totally Legit Software,” and now their system shuts down immediately after login.

The challenge provides a Windows Event Log file and asks us to reconstruct what happened — and more importantly, extract a flag split into three parts. The key idea here is filtering and correlating events by **Event ID** and **timestamp**.

---

## Solution

### 1) Flag Part 1 — Installer activity (Event ID 1033)

I started by creating a **Custom View** in Event Viewer filtering on `Event ID 1033`, which corresponds to Windows Installer activity.

This returned a small set of events. The first one immediately stood out — it referenced software named `Totally_Legit_Software` and contained a suspicious Base64 string.

![Event Viewer showing Event ID 1033 with a Base64 string in the installer log](flag_part1.png)

Decoding the Base64 string in CyberChef revealed the **first part of the flag**. I also noted the timestamp of this event: **11:55:57**.

---

### 2) Flag Part 2 — Registry persistence (Event ID 4657)

Knowing the software name and timestamp, I searched the logs for the keyword `legit` and focused on events occurring around the same time.

This led me to `Event ID 4657`, which logs registry modifications. The event showed a new value being added under the `Run` key — classic persistence behavior.

![Event Viewer showing Event ID 4657 with registry Run key modification and Base64 data](flag_part2.png)

Inside the registry value name was another Base64 string. Decoding it revealed the **second part of the flag**.

---

### 3) Flag Part 3 — Forced shutdowns (Event ID 1074)

For the final piece, I created another Custom View filtering on `Event ID 1074`, which records system shutdown and restart events.

This view only returned four events total, making it easy to inspect each one. One of them contained yet another Base64 string in the shutdown comment field.

![Event Viewer showing Event ID 1074 with Base64-encoded shutdown comment](flag_part3.png)

Decoding this string gave the **final part of the flag**.

---

## Final Flag

Combining all three decoded pieces:

`picoCTF{Ev3nt_vi3wv3r_1s_a_pr3tty_us3ful_t00l_81ba3fe9}`

This challenge is a great reminder that Windows Event Logs can tell a very complete story — if you know which Event IDs to look for.

---

## Tools Used

- Windows Event Viewer (custom views + filtering)
- CyberChef (Base64 decoding)
- Keyword searching and timeline correlation