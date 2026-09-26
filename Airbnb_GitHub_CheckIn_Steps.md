# GitHub Check-In Steps — Airbnb Capstone

Your local clone: `C:\Amit\_Courses\01 Data Analytics -Hero Vired\Capstone Project\Submissions`
Remote repo: https://github.com/amitchcal/Airbnb_data_analysis

This is a manual, offline-capable checklist — everything except the final `git push` step can be done with no internet. Follow it in order.

---

## 1. Copy the files you want to check in

Do **not** work directly inside `Submissions` while actively editing in Power BI Desktop — copy finished/interim files into it at each checkpoint instead. From `Capstone Project\Airbnb Pricing Dashboard\`, copy these into `Capstone Project\Submissions\`:

- `Airbnb_Pricing_Dashboard.pbix` (or whatever the working `.pbix` is currently named) — the WIP dashboard itself
- `Airbnb_Power_BI_Step_by_Step_Guide.md` — the build guide, including the "CURRENT PROGRESS" marker at the top
- `Airbnb_Solo_Submission_Checklist.xlsx`
- `Airbnb_Presentation_Script.md`
- `Airbnb_PowerBI_Project_Plan.xlsx` — the original team plan (submit as-is per the brief)
- `Airbnb_GitHub_CheckIn_Steps.md` (this file, optional)

**Do NOT copy these into the repo, ever:**
- The 3 instructor `.mp4` files (270–385 MB each) — GitHub hard-blocks any file over 100 MB without Git LFS, so a push containing one of these will simply fail.
- The `Dataset` folder's raw CSVs, unless you specifically want the raw data versioned too (check with the instructor whether that's expected or if a data-source citation is enough).
- Anything from the transcription scratch work (audio `.wav` files, transcript `.txt` files) — those were local working files only, not deliverables.

Easiest way to copy: select the files in File Explorer → Ctrl+C → open the `Submissions` folder → Ctrl+V.

---

## 2. Check what's about to be committed (do this every time, don't skip)

Open a terminal (PowerShell or Git Bash) in the `Submissions` folder and run:

```bash
git status
```

Read the output carefully:
- Files listed under **"Untracked files"** are new files git hasn't seen before (your first-time copies).
- Files listed under **"Changes not staged for commit"** are files git already tracks that you've modified (e.g., re-copying an updated `.pbix` or guide over a previous version).
- If anything unexpected shows up here (a file you didn't mean to copy in), delete it from `Submissions` before proceeding.

---

## 3. Stage the files

Stage specific files by name — avoid `git add -A` or `git add .` so nothing sneaks in by accident:

```bash
git add "Airbnb_Pricing_Dashboard.pbix"
git add "Airbnb_Power_BI_Step_by_Step_Guide.md"
git add "Airbnb_Solo_Submission_Checklist.xlsx"
git add "Airbnb_Presentation_Script.md"
git add "Airbnb_PowerBI_Project_Plan.xlsx"
```

(Only add the ones actually present/changed — `git status` from Step 2 tells you which.)

Run `git status` again — staged files now show under **"Changes to be committed."**

---

## 4. Commit

```bash
git commit -m "WIP: data cleaned, star schema + relationships verified, calculated columns and What-If parameters done (Sections 3-7 complete)"
```

Adjust the message to reflect whatever you actually finished by the time you check in — keep it specific (what's done), not generic ("update files").

---

## 5. Push to GitHub

```bash
git push origin main
```

This needs internet access. If you're offline, stop here — commits 3-4 above are already safely saved in your **local** git history; the push can happen later whenever you're back online, with no extra steps (`git push origin main` again will send everything queued up locally).

---

## 6. Verify on GitHub (once pushed)

Open https://github.com/amitchcal/Airbnb_data_analysis in a browser and confirm the files appear with your commit message. If the `.pbix` looks like it didn't upload (some browsers/connections struggle with large binary pushes), run:

```bash
git log --stat -1
```

This shows exactly what was included in the last commit — confirm the `.pbix` is listed with its file size.

---

## Quick reference — the whole sequence in one block

```bash
cd "C:\Amit\_Courses\01 Data Analytics -Hero Vired\Capstone Project\Submissions"
git status
git add "Airbnb_Pricing_Dashboard.pbix" "Airbnb_Power_BI_Step_by_Step_Guide.md" "Airbnb_Solo_Submission_Checklist.xlsx" "Airbnb_Presentation_Script.md" "Airbnb_PowerBI_Project_Plan.xlsx"
git status
git commit -m "WIP: <describe what you actually finished>"
git push origin main
```
