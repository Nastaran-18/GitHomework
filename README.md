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
