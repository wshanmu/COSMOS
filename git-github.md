---
layout: page
title: Git and GitHub Collaboration
description: How student groups share one lab codebase with Git and GitHub.
nav_order: 6
---

# Git and GitHub Collaboration

Use this workflow when your group starts from an instructor-provided GitHub repository, such as the RFID lab repository.

Git helps your group share code without emailing files or overwriting each other's work. GitHub stores your shared copy of the code online.

## Recommended Workflow

Each group should use one shared GitHub fork.

One group member should create the fork and add the other group members as collaborators. Then every group member clones that shared fork to their own laptop.

```text
Instructor repository -> group fork -> each student's laptop
```

Do not push your local changes directly to the instructor repository.

## Step 1: Fork the Starter Repository

For Lab 6, one group member should open:

[https://github.com/wshanmu/RFID_Lab](https://github.com/wshanmu/RFID_Lab)

Then click **Fork**.

This creates a group copy, for example:

```text
https://github.com/GROUP_OWNER_USERNAME/RFID_Lab
```

The group owner should add teammates as collaborators:

```text
GitHub repository page -> Settings -> Collaborators -> Add people
```

Each teammate should accept the invitation before trying to push code.

## Step 2: Clone the Group Fork

Each group member should clone the group fork to their own laptop. Replace `GROUP_OWNER_USERNAME` with the GitHub username of the group member who created the fork.

```bash
git clone https://github.com/GROUP_OWNER_USERNAME/RFID_Lab.git
cd RFID_Lab
```

Check that `origin` points to the group fork:

```bash
git remote -v
```

You should see something like:

```text
origin  https://github.com/GROUP_OWNER_USERNAME/RFID_Lab.git
```

## Step 3: Add the Instructor Repository as Upstream

Each group member should also add the original instructor repository as `upstream`.

```bash
git remote add upstream https://github.com/wshanmu/RFID_Lab.git
```

Check your remotes:

```bash
git remote -v
```

You should see both:

```text
origin    your group fork
upstream  instructor repository
```

Meaning:

- `origin` is your group's shared GitHub repository.
- `upstream` is the instructor's starter repository.

## Step 4: Pull Before You Start Working

Before editing code, update your local copy:

```bash
git checkout main
git pull origin main
```

If the instructor updates the starter repository during the lab, you can also pull from `upstream`:

```bash
git pull upstream main
git push origin main
```

This copies the instructor update into your group fork.

## Step 5: Create a Branch for Your Work

Do not let everyone edit `main` at the same time. Create a branch for your own changes:

```bash
git checkout -b my-branch-name
```

Use short, descriptive branch names, for example:

```bash
git checkout -b distance-plot
git checkout -b touch-detector-thresholds
git checkout -b rfid-log-cleanup
```

## Step 6: Edit Code and Test Locally

Make your changes in VS Code or your preferred editor.

Useful commands:

```bash
git status
```

This shows which files changed.

```bash
git diff
```

This shows what changed inside the files.

Run and test your code before committing. For example:

```bash
python ./plot_single_log.py --help
python ./touch_detector_gui.py
```

## Step 7: Commit Your Changes

After testing, save your work with a commit:

```bash
git add FILE_YOU_CHANGED.py
git commit -m "Describe the change"
```

Example:

```bash
git add plot_multiple_logs.py
git commit -m "Add distance comparison plot"
```

Good commit messages briefly explain what changed:

- `Add KDE plot option`
- `Fix EPC filtering bug`
- `Tune touch detection thresholds`
- `Add per-tag baseline calibration`

Avoid vague messages such as `update`, `stuff`, or `final version`.

## Step 8: Push Your Branch to GitHub

Push your branch to the group fork:

```bash
git push origin YOUR_BRANCH_NAME
```

Example:

```bash
git push origin distance-plot
```

Now your teammates can see your branch on GitHub.

## Step 9: Merge Through a Pull Request

On GitHub, open your group fork. GitHub should show a **Compare & pull request** button for your pushed branch.

Create a pull request from your branch into `main`.

Before merging, another teammate should quickly check:

- Does the code run?
- Does it break other scripts?
- Is the change useful?
- Are there accidental files included?

After the pull request is merged, everyone should update their local `main`:

```bash
git checkout main
git pull origin main
```

## Step 10: Avoid Large or Temporary Files

Do not commit:

- `__pycache__/`
- `.venv/`
- `.DS_Store`
- large raw logs
- temporary screenshots
- random test files

Use your shared Google Slides deck for photos, screenshots, and final plots.

If your group wants to keep small example logs or plots in GitHub, put them in a clearly named folder such as:

```text
example_logs/
figures/
```

Avoid uploading many large raw data files.

## If Two People Edit the Same File

Sometimes Git will report a conflict.

A conflict means two people changed the same part of the same file, and Git does not know which version to keep.

When this happens:

1. Open the conflicted file in VS Code.
2. Look for markers like:

   ```text
   <<<<<<< HEAD
   your version
   =======
   teammate version
   >>>>>>> branch-name
   ```

3. Decide which code to keep.
4. Delete the conflict markers.
5. Test the code again.
6. Commit the resolved file:

   ```bash
   git add conflicted_file.py
   git commit -m "Resolve merge conflict"
   ```

If you are unsure, ask a TA before using commands such as `reset`, `checkout`, or force push.

## Important Safety Rules

Do not use these commands unless a TA tells you to:

```bash
git reset --hard
git push --force
git clean -fd
```

These commands can delete your local work or overwrite teammates' work.

## Minimal Command Cheat Sheet

Check status:

```bash
git status
```

Get latest group code:

```bash
git checkout main
git pull origin main
```

Create a branch:

```bash
git checkout -b my-branch-name
```

Switch branches:

```bash
git checkout main
```

Save changes:

```bash
git add filename.py
git commit -m "Describe my change"
```

Upload branch:

```bash
git push origin my-branch-name
```

Update local `main` after a teammate merges:

```bash
git checkout main
git pull origin main
```

## Key Takeaway

Use GitHub as your group's shared code workspace. Work on separate branches, commit small changes, push often, and merge through pull requests. This prevents teammates from overwriting each other and makes it easier to combine everyone's work into one final version.
