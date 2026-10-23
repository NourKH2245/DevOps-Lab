# Lab sheet 1: Answers
**Team**:
- Nour Imene KHELLOUFI
- Soundes Hiba KORICHI
- Amir El Charif MOHAMMEDI
- Ryad BENMECHRI

## Exercise 1
(a) .gitignore & .git

(b) If we used the command cp –r .../ctf-scoreboard/* instead of cp –r .../ctf-scoreboard/. => It will copy just the visible (non-hidden) files and directories, unlike ‘.’ means everything inside this directory, including hidden files.

(c) .gitignore only works for files that are not tracked yet. If .env is already committed, Git keeps tracking it, and the secret is still in the old commits, so anyone who clones the repo can see it.

## Exercise 2
(a)Git cannot merge these two changes automatically because both branches modified the exact same line in the same file.
Git merge’s algorithm works line-by-line. When the same line was changed differently on both sides, git has no way to know which value is the “correct” one. That’s why git marked the file as being conflicted and halted the merging process and now It’s then the developer’s responsibility to resolve the conflict.

(b)If the two branches changed different lines in the same file, Git can merge them automatically.

(c)It can run in a pre-commit hook on our machine, in a pre-receive hook on the server, or in CI (like GitHub Actions) on each push or pull request. git diff --check can also detect the markers.

## Exercise 3
(a)Git bisect uses a binary search to find the first bad commit between a known good commit and a known bad commit. It took 5 steps to search the 31 commits.

(b)[log2 31] ~= 5, So the number of steps matches the theoretical minimum of binary search. And Bisect cuts the commits in half every step.

(c) What if the script check.sh is inside the repository:  
- Keep the test independent of the history being tested, so Git can reliably compare old and new versions of your project.  
- Git may check out an older commit where the script does not exist or has a different version. Keeping it outside the repository ensures the same test runs at every commit.

(d) What if the 31 commits are squashed into one commit: 
Git identifies the single squashed commit as bad but cannot determine which original change introduced the bug because the intermediate commits are gone.

## Exercise 4
(a) The exit code for each scan: 
- Gitleaks dir . : 0 no secrets detected in the current files 
- Gitleaks git –v . : 1 secret detected in Git history (leak found) 

(b) The rule that matches is aws-access-token. 

(c) Why does the second commit remove nothing from history: 
Every Git commit stores a reference to its parent commit. The second commit points to the first commit, which still contains the secret. Restoring the old configuration file removes the secret only from the current version, not from previous commits. The secret remains recoverable from Git history, and only rewriting the history removes it.

(d) Someone pushed a real key. The first 2 steps that should be taken are: 
	1. Revoke or change the key, assume everyone knows it. 
	2. Rewrite history using git filter-repo, force push and coordinate the cleanup with the team.

## Exercise 5
(a) Why is branch protection also a security control? 
Branch protection prevents unauthorized or unreviewed changes from being pushed directly to main. Requiring pull requests and approval ensures another team member checks the changes before merging. Blocking force pushes and deletion protects the branch history from being overwritten or removed. 

(b) Name the attack that makes it harder. 
History rewriting or unauthorized code injection into the main branch. It also makes accidental or malicious deletion of the branch harder. 

(c) Why must the repository be public on a free GitHub account? 
For this exercise, the required branch-protection rules may depend on GitHub's plan and repository type. The assignment uses a public repository because the required protection features are available for public repositories on GitHub Free, whereas some rules for private repositories require a paid plan.

## Conclusion
This practical work highlighted the role of Git and GitHub in DevOps by improving collaboration, code management, debugging, and security throughout the development process.
