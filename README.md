# GitHomework

Name: Nastaran Rajabi
Student Number: 40231453

Question 1: Difference between merge and rebase command
git merge - keeps both line of work as they are and ties them together with a new merge commit that has two parent. Nothing that already exists is touched, so the history stays a branching graph and shows where the branches split and where they came back together.
git rebase - works by moving your branch. Git finds the commits that are only on your branch, sets them aside, moves the branch to the tip of the target, and replays the commits one at a time on top. The replayed commits have the same changes but they are new commits with new hashes, and the originals are abandoned. The result is a straight line with no merge commit.
Where I would use merge? git switch main / git merge --no-ff feature-login
--no-ff forces a merge commit so the feature stays visible in history. Merge rewrites nothing, so it is safe for a branch other people already have.

Use rebase to update my own feature branch with the latest main before a pull request:
git fetch origin
git rebase origin/main
On conflict: fix, git add, git rebase --continue (or git rebase --abort to cancel). If the branch was already pushed, use git push --force-with-lease, which refuses to overwrite commits someone else added. Rebase fits because the branch is only mine and the history stays clean. Never rebase commits others have pulled.
Question 2: Staging part of a file

A hunk is one continuous block of changes in git diff, starting with a header like @@ -10,6 +10,8 @@.

bash
git add -p README.md

-p (--patch) shows each hunk and asks what to do:

y stage it, n skip it
s split it into smaller hunks
e edit it manually, line by line
a / d stage / skip this and all remaining hunks in the file
q quit, ? help

Then check and commit:

bash
git diff --staged
git commit -m "Fix typo in installation section"

Only the chosen hunks are committed. The rest stay unstaged in the working tree.

Benefit: unrelated changes in one file (a bug fix and a cleanup) become separate, focused commits that are easier to review, revert, and debug.

Question 3: git bisect

It does a binary search through history to find the first bad commit.

bash
git bisect start
git bisect bad            # current commit is broken
git bisect good v1.0      # this one worked

Git checks out a middle commit. I test it and answer:

bash
git bisect good    # bug not here, so it came later
git bisect bad     # bug is here, so it came earlier or at this commit
git bisect skip    # can't test this commit

Each answer halves the range until git prints <hash> is the first bad commit. Then:

bash
git bisect reset   # return to my original branch

Automation: git bisect run ./test.sh (exit 0 = good, 1–127 = bad, 125 = skip).

Example: with 200 commits between v1.0 and HEAD, bisect needs about log₂(200) ≈ 8 tests instead of up to 200.

Why useful: testing effort grows logarithmically, and the result is the exact commit that introduced the bug.

Question 4: Checking changes before committing

1. Checking state

bash
git status          # README.md appears twice: staged and not staged; tmp/ is untracked
git diff            # working tree vs. staging: edits made after git add
git diff --staged   # staging vs. last commit: what will be committed

git add takes a snapshot of the file at that moment. Staged changes are that snapshot; unstaged changes are the edits made afterwards.

2. Plain git commit (no -a)

Only staged changes are committed, because a commit is built from the staging area, not the working tree. The later edits stay in the working tree as unstaged changes.

3. Ignoring tmp/

Add this line to .gitignore in the repo root:

tmp/

(For a personal-only rule, use .git/info/exclude.)

This is not enough for a file that is already tracked, because .gitignore only affects untracked files. Stop tracking it with:

bash
git rm --cached <file>
git commit -m "Stop tracking <file>"

--cached removes it from tracking but keeps the file on disk. After that, the .gitignore rule applies.

Question 5: Resolving a merge conflict

1. Why it happened

On main I ran git merge feature-description. Both branches changed the same line of README.md to different text, and git cannot know which is right. Not every change causes a conflict: edits to different files, or to different parts of one file, merge automatically.

2. Finding conflicts and markers

bash
git status        # files under "Unmerged paths" (both modified)
git diff          # shows the conflicting regions

In the file:

<<<<<<< HEAD
version from main
=======
version from feature-description
>>>>>>> feature-description

The markers show where each version starts and ends. They are not real content and must be removed.

3. Keeping both changes

Edit README.md so the text contains the information from both versions.
Delete the three marker lines.
Check none remain: grep -nE '^(<<<<<<<|=======|>>>>>>>)' README.md (no output = clean).
git add README.md
git merge --continue (or git commit)
Verify with git status (clean) and git log --graph --oneline.

4. Cancelling

bash
git merge --abort

This restores the state from before the merge. Nothing is lost, since the working tree was clean.

Question 6: Undoing changes

1. Discard unstaged edits in notes.txt

bash
git restore notes.txt

It copies the file from the staging area (same as the last commit) into the working tree. The edits are lost permanently, since they were never saved anywhere. Staging, history, and other files are unaffected.

2. Unstage notes.txt, keep edits

bash
git restore --staged notes.txt

--staged makes the staging area the target, resetting that file to match HEAD. The working tree keeps my edits, and history is unchanged.

3. Undo a pushed bad commit on shared main

bash
git revert <hash>
git push origin main

revert creates a new commit that applies the opposite changes. The working tree loses that commit’s changes, staging ends clean, and history keeps the bad commit and gains a Revert "..." commit. A normal push works with no force. (-m is only needed for merge commits; --no-edit skips the message editor.)

Deleting the commit with git reset --hard plus git push --force is a bad idea because it rewrites history teammates already have. Their copies would diverge from the remote, causing confusing conflicts, and the force push could erase commits others pushed afterwards.
