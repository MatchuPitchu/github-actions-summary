# Git

- a (free) `version control system` to manage source code changes and easily roll back to older code snapshots or develop new features without breaking production code
- save code snapshots: `commits`
- work with alternative code versions: `branches`
- move between branches & commits: `checkout`

## Basics

- set default editor & global config: `git config --global core.editor "code --wait"`
- set to open config in default editor: `git config --global -e`
- set global config `End of Line` (EoL): `git config --global core.autocrlf true`

- `git init`: create repository in a folder
- `git add [<file(s)> | -A]`: stage changes in certain files/folders or all changes in repository for next commit
- `git commit -m "your message"`: create a commit that includes all staged changes
- `git status`: current state of git repository
- `git log`: list commit history
  - `git log --graph --oneline --decorate`
- `HEAD`: a pointer that controls which commit/snapshot is currently loaded
  - `git checkout <commitId>`: temporarily move to another commit
  - `git checkout <branch name>`: return to latest snapshot of a branch
- `git revert <commitId>`: undoes changes of a commit by creating a new commit
- `git reset --hard <commitId>`: undoes changes by deleting all(!) commits since `<commitId>` and deletes whole history after `<commitId>`

- `.gitignore`: file containing files and folders that should not be included in the git repository

- `git remote add origin <GitHub-URL>`: connect remote repo (-> `origin` is naming convention) a local repo
- `git push --set-upstream origin <branch name>`: set a connection between remote repo and a branch
- `git push`: send local commits to `origin` (-> remote repo)

- `git pull`: downloads (-> `fetches`) latest snapshot of the remote repository and integrates it into the current branch

- `git clone <GitHub-URL>`: clone a remote repository on your local machine

- `git stash`: Änderungen in Arbeitskopie werden zwischengespeichert
  - `git stash -m "YOUR_TEXT"`: alle Änderungen zwischenspeichern unter bestimmtem Namen
  - `git stash push -m "YOUR_TEXT" <file>`: nur bestimmte Datei unter bestimmtem Namen zwischenspeichern
  - `git stash list`: Liste von zwischengespeicherten Änderungen
  - `git stash apply stash@{NR_AUS_LISTE}`: Änderungen mit bestimmtem Namen aus Zwischenspeicher wieder aufspielen
  - `git stash pop`: Änderungen aus Zwischenspeicher später wieder aufspielen
  - `git stash drop stash@{NR_AUS_LISTE}`: Änderungen aus Zwischenspeicher löschen

## Branches

- `git branch --list`: list all branches
- `git branch -d <name>`: delete branch
- `git switch -c <new-branch>`: create branch based on current snapshot of current branch where you are located and switch to it
  - older ways:
    - `git branch <new-branch>`: create branch based
    - `git checkout -b <new-branch>`: create branch and switch to it
  - naming conventions: e.g. `feature/blog`, `fix/modal`
- `git switch <name> (-)`: switch to another branch OR to last branch with `-`
  - older way: `git checkout <name>`
- `git merge <name branch x>`: merge commits of `branch x` into current branch and append these new commits to commits in your current branch

## Collaboration

- if you have a public repo on `GitHub`, you can invite other people to collaborate (-> `Settings` -> `Collaborators`)
- important: protect branch(es) like `main` branch (-> `Settings` -> `Branches`) to avoid editing or deleting

- `Pull Request`: ask pull request to be allowed to merge your branch into another remote branch on `GitHub` or `GitLab`

- `Fork`: is a copy of a repository; allows you to freely experiment with changes without affetcting the original project
  - if you add new commits to your `fork`, you can then do a `Pull Request` in original repo on `GitHub` and chose option `compare across forks`, i.e. set your fork repo branch as origin for a comparison of the `Pull Request`
