<div align="center">

# 🔓 John the Ripper: Cracking a Locked PDF, Plus a Lesson in Checking Your Files

**Week 3, Project Module 1 — cracking `My Locked PDF1.pdf` with `pdf2john` and `john` in Kali Linux, and how a mixed-up file name nearly gave me the wrong answer**
</div>

<p align="center">
  <img src="https://img.shields.io/badge/Tool-John%20the%20Ripper-1679A7?style=flat-square" />
  <img src="https://img.shields.io/badge/Role-Pentester%20(Password%20Auditing)-C00000?style=flat-square" />
  <img src="https://img.shields.io/badge/Password-good--luck-C8570A?style=flat-square" />
  <img src="https://img.shields.io/badge/Phase-Password%20Cracking-238F89?style=flat-square" />
  <img src="https://img.shields.io/badge/NetworkWalks-B082-C00000?style=flat-square" />
</p>

---

## 📌 Project Overview

This is **Week 3, Project Module 1 (W3-PM1): Password Cracking with JTR** from the Networkwalks Cybersecurity internship. The task is to crack the password of a locked PDF, `My Locked PDF1.pdf`, using John the Ripper (JTR), the lab's official route being the command-line `john` tool and the Johnny GUI on Windows.

The lab guide is written for Windows and uses Johnny. I did the whole task on **Kali Linux** instead, using `pdf2john` and the command-line `john` (the lab guide itself notes Kali users can open JTR directly, since it's pre-installed). I did not use the Johnny GUI.

Along the way I ran into a genuine mix-up that is worth documenting honestly: I have **two different files that both start with "My Locked PDF1"** in the same folder — the one actually attached to this lab (`My Locked PDF1.pdf`) and a different one left over from another module, **W3-PM2 (Password Cracking with NW Tools)**, named `My Locked PDF1 (for NW Tools Lab WK3-PM2).pdf`. I first extracted a hash from the wrong one by mistake, using the lab's own browser-based method, and only caught it by comparing two different cracked passwords. The correct password for this lab's actual target file, confirmed by opening it, is **`good-luck`**.

As a bonus, I also cracked two extra locked PDFs I had for further practice (`My Locked PDF2.pdf` and `My Locked PDF3.pdf`), not part of the required task.

| 🔎 Item | ✅ Result |
| ------- | --------- |
| Tool | John the Ripper (`pdf2john` + `john`), Kali Linux |
| Target (required task) | `My Locked PDF1.pdf` |
| **Correct password** | **`good-luck`** (confirmed by opening the file; flag: `nw{cybersecurity_flag_captured_2608}`) |
| Mix-up | `My Locked PDF1 (for NW Tools Lab WK3-PM2).pdf` — a different module's file, cracked to `password1`, which is **not** this lab's answer |
| Bonus crack 1 | `My Locked PDF2.pdf` → `password1` (flag: `nw{networkwalks_persistence_jtr_270521}`) |
| Bonus crack 2 | `My Locked PDF3.pdf` → `1qaz2wsx` (flag: `nw{networkwalks_flag_260821_1}`) |

---

## 🎯 Objectives

- Extract a crackable hash from a password-protected PDF
- Run John the Ripper against that hash and read the cracked password
- Understand what `pdf2john` actually does, and that it reproduces exactly what the lab's browser-based hash extractor does
- Catch and correctly explain a mix-up between two similarly-named files
- Confirm a cracked password by actually opening the file with it
- Practice the same process on two further locked PDFs

---

## 🛡️ Why This Matters for Security

Password cracking tools like John the Ripper exist to test how strong a password really is, not just to "break in". A password that falls to a basic wordlist attack in under a second, like all three PDFs in this write-up, tells a defender that whatever protected that file would not have stood up to a real attacker either.

The mix-up in this lab is its own security lesson. In a real engagement, hashing or testing the wrong artifact (the wrong file, the wrong host, the wrong account) produces a result that looks completely valid, it's just answering the wrong question. The only way I caught it here was by noticing two different tools gave two different, both internally consistent, answers, and then checking which file each one had actually touched.

> ⚠️ **Important:** The files cracked here are training PDFs supplied as part of the Networkwalks course, for the sole purpose of learning password auditing. The same tools should never be run against a password-protected file you do not own or do not have permission to test.

---

## ⚙️ Lab Configuration

| 🧩 Component | ⚙️ Configuration |
| ------------ | ---------------- |
| 🖥️ Analyst PC | Kali Linux VM (VirtualBox), plus the host machine's browser for the lab's hash-extractor website |
| 🛠️ Tools | `pdf2john` (ships with John the Ripper), `john` (command line); Johnny GUI not used |
| 🎯 Target | `My Locked PDF1.pdf` (required), plus `My Locked PDF2.pdf` and `My Locked PDF3.pdf` (bonus) |
| 📁 Files present | `My Locked PDF1.pdf`, `My Locked PDF1 (for NW Tools Lab WK3-PM2).pdf`, `My Locked PDF2.pdf`, `My Locked PDF3.pdf`, each a separate file |
| 🔒 Privacy | Kali's own hostname (`kali`) is left as is; the host machine's hostname is blacked out where it appears |
| 🏆 Scoring | None; each PDF opens to its own "Congratulations" flag page instead of a result card |

---

# 🪜 Lab Procedure

## Step 1. Find the target file and extract its hash
`My Locked PDF1.pdf` was sitting in my Documents folder.

![PDF1 seen in Documents](screenshots/w3pm1-01-pdf1-seen-in-documents.png)

I ran `pdf2john` on it directly in Kali, which pulls the crackable hash straight out of the PDF:

```bash
pdf2john "My Locked PDF1.pdf" > hash1.txt
```

When I then moved into my `pm1` working folder and checked the `hash1.txt` already saved there, it held a **different** hash value from the one I had just generated. That was my first sign something didn't line up.

![pdf2john output vs. an already-saved hash](screenshots/w3pm1-02-pdf2john-vs-already-saved-hash.png)

## Step 2. Crack both hashes and notice the conflict
I ran `john` on the saved `pm1/hash1.txt` first, then went back and ran it on the hash I had just extracted directly from the PDF with `pdf2john`:

```bash
john hash1.txt
```

The saved hash cracked to **`password1`** (shown with no filename label, just `(?)`, because that hash file had no filename prefix). The hash I extracted myself cracked to **`good-luck`**, and this time John showed the label `(My Locked PDF1.pdf)` because `pdf2john` writes the filename in front of the hash.

![Two different cracked passwords](screenshots/w3pm1-03-two-different-cracked-passwords.png)

**Two different, both genuinely cracked, passwords for what I thought was the same file. This is where I had to stop and work out why.**

## Step 3. Trace the saved hash back to its source
I went looking for how `pm1/hash1.txt` had been produced. Following the lab's own Step 3, I had used the browser-based extractor at onlinehashcrack.com to get a hash without touching the command line.

![Searching for John/Johnny on the VM](screenshots/w3pm1-04-host-vm-window-search-john.png)

![OnlineHashCrack landing page](screenshots/w3pm1-05-onlinehashcrack-landing.png)

![Upload form](screenshots/w3pm1-06-onlinehashcrack-upload-form.png)

Here is the mistake: the file I had actually uploaded was **`My Locked PDF1 (for NW Tools Lab WK3-PM2).pdf`**, not `My Locked PDF1.pdf`. That other file belongs to a different module, **W3-PM2 (Password Cracking with NW Tools)**, which was sitting in the same downloads folder with a confusingly similar name.

![Wrong file uploaded to the extractor](screenshots/w3pm1-07-onlinehashcrack-wrong-file-uploaded.png)

I copied that hash into a `hash1.txt` file to use with Johnny/John, as the lab describes.

![Creating hash1.txt on the host](screenshots/w3pm1-08-host-touch-hash1.png)

![Pasted hash matches the wrong file's extraction](screenshots/w3pm1-09-host-hash1-pasted-matches-wrong-file.png)

## Step 4. Confirm what pdf2john actually does
Before trusting the command-line result over the browser one, I wanted to understand what `pdf2john` was actually doing, and whether it was a reliable substitute for the lab's browser-based method. I asked an AI assistant to explain it.

![What is pdf2john](screenshots/w3pm1-10-ai-what-is-pdf2john.png)

![How it compares to the lab guide's browser method](screenshots/w3pm1-11-ai-how-it-compares-to-lab-guide.png)

The answer: the OnlineHashCrack website is, in its own words, just running `pdf2john` on their server, and since Kali already has that same script installed, running it locally gives the identical extraction without uploading the file anywhere.

![How to run pdf2john in Kali](screenshots/w3pm1-12-ai-how-to-run-pdf2john-in-kali.png)

## Step 5. Confirm the wrong file really does open with password1
To be sure `password1` wasn't simply wrong, I opened the file I had actually hashed, `My Locked PDF1 (for NW Tools Lab WK3-PM2).pdf`, with that password.

![Password prompt for the wrong file](screenshots/w3pm1-13-wrong-file-password-prompt.png)

It opened. The flag it shows, `networkwalks_flag1_pr_070521_1`, belongs to the W3-PM2 module, not this one.

![Wrong file opens, but it's a different module's flag](screenshots/w3pm1-14-wrong-file-opens-different-module-flag.png)

**So `password1` is a real, correctly cracked password — just for the wrong file.**

## Step 6. Ask why the same-looking task gave two different answers
I asked the AI assistant directly why the two commands produced two different passwords.

![Asking why two passwords](screenshots/w3pm1-15-ai-question-why-two-passwords.png)

![Both hashes were cracked successfully](screenshots/w3pm1-16-ai-answer-both-hashes-real.png)

![Explanation: a PDF's user vs. owner password, and different source files](screenshots/w3pm1-17-ai-answer-user-vs-owner-password.png)

The AI first raised a general possibility (that a single PDF stores two separate passwords, a user/open password and an owner/permissions password, and John might find either one). But the real reason here, given the evidence, was simpler: **the two commands ran against two different physical files that just happened to share most of their name.**

![Why re-running john shows "no password hashes left to crack"](screenshots/w3pm1-18-ai-answer-pot-file-cache.png)

## Step 7. Confirm the real answer
With the mix-up explained, I went back to the hash I had extracted with `pdf2john` directly from the actual `My Locked PDF1.pdf`, cracked to `good-luck`, and opened that file with it.

![PDF1 opens with good-luck](screenshots/w3pm1-19-pdf1-confirmed-with-goodluck.png)

It opened, with the flag `nw{cybersecurity_flag_captured_2608}`.

**Answer: `good-luck` is the correct password for `My Locked PDF1.pdf`, confirmed by opening the file.**

---

# 🎁 Bonus Cracks: PDF2 and PDF3

These two PDFs were not part of the required task. I had them for further practice and ran the same `pdf2john` + `john` process on each, directly in Kali.

## Bonus 1: My Locked PDF2.pdf
```bash
pdf2john "My Locked PDF2.pdf" > hash2.txt
john hash2.txt
```

![Starting the crack on PDF2](screenshots/w3pm1-20-bonus-pdf2-crack-start.png)

![PDF2 cracked to password1](screenshots/w3pm1-21-bonus-pdf2-cracked-password1.png)

Cracked password: **`password1`**. I confirmed it by opening the file.

![Password prompt for PDF2](screenshots/w3pm1-22-bonus-pdf2-password-prompt.png)

![PDF2 flag captured](screenshots/w3pm1-23-bonus-pdf2-flag-captured.png)

Flag: `nw{networkwalks_persistence_jtr_270521}`.

## Bonus 2: My Locked PDF3.pdf
```bash
pdf2john "My Locked PDF3.pdf" > hash3.txt
john hash3.txt
```

![Starting the crack on PDF3](screenshots/w3pm1-24-bonus-pdf3-crack-start.png)

![PDF3 cracked to 1qaz2wsx](screenshots/w3pm1-25-bonus-pdf3-cracked-1qaz2wsx.png)

Cracked password: **`1qaz2wsx`** (a keyboard-walk pattern). Confirmed by opening the file.

![Password prompt for PDF3](screenshots/w3pm1-26-bonus-pdf3-password-prompt.png)

![PDF3 flag captured](screenshots/w3pm1-27-bonus-pdf3-flag-captured.png)

Flag: `nw{networkwalks_flag_260821_1}`.

---

# 📊 Results

| # | 🎯 File | ✅ Password | 🔎 How I Found It | 🏁 Confirmed By |
| - | ------- | ----------- | ------------------ | --------------- |
| 1 | `My Locked PDF1.pdf` (required task) | **`good-luck`** | `pdf2john` + `john`, direct on the correct file | Opened the PDF; flag `nw{cybersecurity_flag_captured_2608}` |
| — | `My Locked PDF1 (for NW Tools Lab WK3-PM2).pdf` (mix-up, not this lab) | `password1` | Browser extractor (wrong file uploaded by mistake) | Opened the PDF; flag belongs to module W3-PM2 |
| 2 | `My Locked PDF2.pdf` (bonus) | `password1` | `pdf2john` + `john` | Opened the PDF; flag `nw{networkwalks_persistence_jtr_270521}` |
| 3 | `My Locked PDF3.pdf` (bonus) | `1qaz2wsx` | `pdf2john` + `john` | Opened the PDF; flag `nw{networkwalks_flag_260821_1}` |

*Footnote: the lab has no result card or score; each correct password is confirmed by the PDF actually opening and showing its own flag text.*

### Timeline (from screenshot timestamps)
| ⏱️ Time | 🔄 What happened |
| ------- | ---------------- |
| 16 Nov, 10:56 | Found `My Locked PDF1.pdf` in Documents |
| 16 Nov, 11:25 | `pdf2john` run; noticed the saved `hash1.txt` differed |
| 16 Nov, 11:29 | Both hashes cracked: `password1` and `good-luck` |
| 16 Nov, 16:56–17:06 | Retraced the browser-extractor steps; found the wrong file had been uploaded |
| 16 Nov, 17:15–17:30 | Opened the wrong file with `password1`; asked the AI assistant why the two methods diverged |
| 16 Nov, 17:37–17:38 | AI explained the real cause (two different files) and the `.pot` cache behaviour |
| (confirmation) | Opened the real `My Locked PDF1.pdf` with `good-luck` |
| 17 Nov, 04:18–04:20 | Bonus: PDF2 and PDF3 cracked and confirmed |

---

# 🔭 Observations Beyond the Task Questions

1. **`pdf2john` and the lab's browser extractor are the same tool.** The website is running `pdf2john` server-side; running it locally in Kali gives the identical hash, without uploading the file anywhere.
2. **A cracked password is only as correct as the file you hashed.** John the Ripper did its job correctly both times; the only "error" was mine, picking the wrong file from a folder of similarly-named PDFs.
3. **This course appears to reuse a small set of passwords across modules.** `password1` cracked both the wrong file here and `My Locked PDF2.pdf`; `good-luck` and `1qaz2wsx` were each unique to one file.
4. **Re-running `john` on an already-cracked hash shows "No password hashes left to crack".** John remembers cracked hashes in `~/.john/john.pot` and will not repeat work; `john --show hash1.txt` is the way to see a password it already found.
5. **All three real locked PDFs fell in under a second to John's default single-crack mode**, using nothing more than the standard `password.lst` wordlist. None of these passwords would survive real-world use.

---

# 🐞 Challenges Encountered

* **Issue: uploaded the wrong file to the hash extractor.** I grabbed `My Locked PDF1 (for NW Tools Lab WK3-PM2).pdf` instead of `My Locked PDF1.pdf` when using the browser-based tool in Step 3 of the lab, because the two files sit in the same folder with almost the same name. The hash this produced was completely genuine, just for the wrong target, and it cracked to a password (`password1`) that even opened a real PDF, which very nearly let the mistake pass unnoticed.
  **Resolution:** I ran `pdf2john` directly on `My Locked PDF1.pdf` as well, compared the two cracked passwords, traced the saved hash back to the website upload step, and confirmed the correct answer (`good-luck`) by opening the actual target file.
  **What I'll do differently:** Rename or move files with confusingly similar names before starting a task, and always confirm a cracked password by opening the actual intended file, not just by seeing "Session completed" in the terminal.

* **No other blockers.** `pdf2john` and `john` ran cleanly on Kali for all three PDFs, with no installation needed.

---

# 💡 Key Learnings

1. A password cracker can only tell you it matched a hash, not that you hashed the right thing. Checking the filename, every time, is part of the job.
2. `pdf2john` is not a separate method from the lab's browser tool, it's the exact same extraction, just run locally instead of uploaded.
3. Two internally consistent, both-correct-looking results can still mean one of them answers the wrong question. The fix was comparing them side by side, not trusting the first one that worked.
4. John's `.pot` file means cracked passwords are remembered. `john --show` is how to recall one without a fresh crack.
5. None of these three passwords (`good-luck`, `password1`, `1qaz2wsx`) would resist a real attacker for more than a second. That's the whole point of the lab: weak passwords on sensitive files are not a theoretical risk.
6. Confirming a cracked password by actually opening the file is worth the extra minute. It's what caught this mix-up.

---

# 🔐 Security & Ethical Use

This work is for education only. All files cracked here are training PDFs supplied by Networkwalks for this course, with no real sensitive data inside them. Password cracking tools like John the Ripper should only ever be run against your own files, or files you have explicit written permission to test.

---

# 🔗 Tools & Resources

- **John the Ripper:** https://www.openwall.com/john/
- **pdf2john:** ships with John the Ripper
- **NetworkWalks:** https://networkwalks.com

---

# 👤 Author
**Philip Ajik**
Cybersecurity Student (Batch B082) — Networkwalks Academy
LinkedIn: https://www.linkedin.com/in/philip-ajik-a22944384

---

## 📌 Project Information
**Program Name:** Cybersecurity at Networkwalks | **Task:** W3-PM1 | **Project:** Password Cracking with JTR | **Repository:** GitHub
