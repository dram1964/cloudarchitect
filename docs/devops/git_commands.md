# Getting a Repository
Turn an existing local directory into a git repository:

`git init`

Add a **.gitignore** to exclude files from tracking by Git
Clone a repository with:

`git clone <repository_url>`

Clone takes a full copy of all the data the remote repository has. 
To see all the branches of the repository, use:

`git branch -a`


# Recording Changes
To start tracking changes to a file, use:

`git add <filename>`

To see what you've changed but not yet staged:

`git diff`

To see what you've staged but not yet committed:

`git diff --staged`

To unstage a file

`git reset HEAD <filename>`

Or

`git restore --staged <filename>`

Commit staged changes with a commit message:

`git commit -m "Commit message"`

To amend the previous commit, stage the additional files and then use:

`git commit --amend`

To discard changes to a file since the last commit:

`git checkout -- <filename>`

Newer version of above command

`git restore <filename>`

To remove a file from the staging area and your local directory:

`git rm <filename>`

To remove a file from the staging area but keep it in your local directory:

`git rm --cached <filename>`

To remove all files matching a pattern (the wildcard needs to be escaped with backslash):

`git rm data/\*.csv`

To rename a file:

`git mv <filename.old> <filename.new>`


# Viewing Changes
To view the history of commits:

`git log`

To view the changes associated with the last two commits:

`git log -p -2`

To view changes for a particular file:

`git log -- <path_to_file>`

To view all commits for current branch with one line per commit:

`git log --oneline`

To view all commits for all branches with one line per commit:

`git log --oneline --all`


# Working with Remotes
You can query the remote servers configured for your local copy with:

`git remote -v`

For a single remote you will receive two URLs: one for **fetch** and one for **push**
After cloning a repository the default remote is named **origin**. You can add additional
remotes:

`git remote add <abbreviaion> <URL>`

You can then fetch from the new remote:

`git fetch <abbreviaion>`

You can then checkout the new remote or merge it into one of your branches

`git checkout abbreviation/master`

The fetch command pulls all the data from the specified remote. `git fetch origin`
will pull any new data from the origin since you last cloned it. Fetch will not 
automatically merge the data with your own work. If your current branch is set up to track a 
remote branch `git pull` will fetch and merge that branch to your current branch
To push your changes use

`git push origin master`

This will not work if someone else has pushed to origin/master since you cloned it. You will
first need to fetch their work and merge this into yours before you can push successfully
To inspect the remote branch, use:

`git remote show origin`

The output of this command will show you:

- which local branches are tracking which remote branches
- which remote branches you don't currently have
- which local branches no longer exist on the remote (stale branches)

Running `git fetch origin` will collect the remote branches that you don't currently 
have and `git remote prune` or `git remote prune origin` to remove stale branches.

# Tagging
To add a tag to the current commit:

`git tag -a v0.1 -m "Tag message"`

Add a tag to a specific commit

`git tag -a v0.2 <commit checksum>`

List tags

`git tag`

Tags are not automatically transferred to the remote. They need to be explicitly mentioned:

`git push origin v0.2`

To push all tags:

`git push origin --tags`

Tags can be removed using:

`git tag -d v0.2`

To delete the tag from a remote use:

`git push origin --delete <tagname>`

Show the commit annotated with a specific tag:

`git show v0.1`


# Branches
Commits are pointers to the snapshots for the objects that had been staged. The commit also contains commit metatdata (author, email, committer) and a pointer to the previous commit. The pointer 
points to the root tree for the commit, which contains pointers to the blobs stored with the commit. A branch in Git is simply a pointer to a commit. Each time you commit on a branch, the branch 
pointer moves forward to the current commit. Creating a new branch, creates a new pointer to the 
current commit. A special pointer called **HEAD** is maintained to represent the current 
branch that you are on. 
There are multiple ways to create and switch to a new branch. This can be done in two steps:

        git branch <branchname>
        git checkout <branchname>

Or either of the following can be used:

- `git checkout -b <branchname>`
- `git switch -c <branchname>`

To switch to an existing branch use either one of:

- `git checkout <branchname>`
- `git switch <branchname>`

New branches will have their commit pointer pointing to the same commit as the branch it 
was created from. When new commits are added to the branch, the pointer is moved on to a 
new commit. To bring these commits into another branch you can use the merge subcommand

    git switch -c feature123
    ### edit and commit some files
    git switch main
    git merge feature123

Providing no changes have been made in **main** the git merge will create a fast-forward
merge: where the commits from the merged branch are added to the commits on the **main**
branch and the pointer from **main** is moved to the last commit from the merged branch.

If the **main** branch has commits after the branch to be merged was created, the current
branch of **main** is no longer a direct ancestor of the commits on the branch to be merged-in.
In this case, Git creates a new snapshot that points to the latest commits on both branches and 
incorporates the changes from both branches. This is known as a merge commit.

Occaisionally, conflicts will occur during merge commits. This happens when both branches have
made changes to the same file. In this case, `git merge` will report the conflict and
the merge commit will be paused. Run `git status` to see which files need manual 
attention. The file will contain the conflicting lines from both files. After the conflicts have 
been resolved, the merge commit can be completed using `git add` followed by
`git commit`. 

The `git branch` subcommand contains lots of useful options for managing your branches. The 
`--merged` switch shows branches that have been merged into the current branch and 
the `--no-merged` command shows branches that have not yet been merged in. To 
delete a branch, use:

`git branch -d <branchname>`

If the branch to be deleted contains work that has not yet been merged in, the command will 
fail. You can still force the delete using:

`git branch -D <branchname>`

Use the `--move` switch to rename a branch:

`git branch --move <old_name> <new-name>`

This will change the name of the local copy of the branch. To replicate this on the 
server you will need to both push the new branch name and delete the old branch name on the 
remote:

    git push -u origin <new-name>
    git push origin --delete <old_name>

Care should be taken when re-naming and deleting remote branches that may be in use or ancestors
for other branches held locally elsewhere. Also, if pipelines depend on the branch to be renamed, 
these scripts will also need to be adjusted for the new name
Remote tracking branches are local references to the state of remote branches at the time that
you last pulled them. To synchronise your remote tracking branches use:

`git fetch <remotename>`

When you want to share a local branch, you can push that up to the server:

`git push <remotename> <branchname>`

This will allow others to pull the branch from the server using:

`git fetch origin`

To create a local copy of the new branch, you will need to checkout the branch:

`git checkout -b <branchname> <origin/branchname>`

If the local branch does not already exists and exists on only one remote, 
this command can be shortened to:

`git checkout <branchname>`

The `-b` option allows you to specify an alternative name for the tracking branch
To see the difference between your local tracking branches and the remote branches at the
time you last fetched them, use:

`git branch -vv`

To see the difference between your local tracking branches and the current state of the remotes, 
you will need to fetch them first:

    git fetch -all 
    git branch -vv

Fetching a remote tracking branch will not update your local working directory. You will need 
to merge the remote tracking branch into your working directory manually:

`git merge <origin/branchname>`

Alternatively `git pull` does both a fetch and a merge

# Rebasing
When two branches have divergent commit histories, then a merge commit will create a new 
snapshot from the most recent commits on both branches and the most recent common ancestor commit for both branches. An alternative approach is the available using the `rebase` command. 
Rebase will:

- stash away the divergent commits from the current branch
- rewind the current branch to the common ancestor for both branches
- apply the commits from the source branch
- re-apply the commits that were previously stashed away

To rebase the **feature1** branch to the main branch:

    git checkout feature1
    git rebase main


The **feature1** branch will now have the commit history from **main**, 
with the additional commits from **feature1** appended. When you are ready to merge 
**feature1** back to **main** a fast-forward merge will suffice:

    git checkout main
    git merge feature1

Rebasing is typically used to where you want to cleanly apply your commits to a remote branch. 
You can rebase your local branch to the remote branch and then create a pull request, so that
when the request is approved, the new commits can be applied without fear of merge conflicts.
Rebasing whilst keeping all the changes applied in a branch will result in new commits being 
created when the original commits are re-applied. This could cause a problem if someone else 
has created work based on your original branch. The simple lesson from all this is **never rebase anything that you've pushed somewhere else.**

# Reversing Changes
HEAD is the symbolic name for the currently checked out commit. HEAD is normally attached to a branch
but you can attach it to a commit:

`git checkout commit_hash`

Since commit hashes are awkward to work with, you can use relative refs to move around: 

- Use the carat symbol to move up one commit at a time
- Use the tilde symbol followed by an integer to move upwards that integer number of time

Thus you can move HEAD using relative refs. The following code moves HEAD back two commits before the $current_commit:

    git checkout $current_commit
    git checkout HEAD^
    git checkout HEAD^

The same effect can be produced with:

`git checkout HEAD~2`

Instead of moving HEAD, you can reassign a branch to to a previous commit:

`git branch -f main HEAD~3`

This will move the main branch to three commits behind HEAD. Or you can move the branch forward to a specific commit hash:

`git branch -f main $specific_commit`

You can reverse changes by moving a branch reference backwards in time to an older commit:

`git reset HEAD~1`

'reset' works for local branches on your own machine. To reverse changes for remote branches 
that others are using, use 'revert':

`git revert HEAD`

'revert' appends a new commit that has the effect of undoing the previous commit. This commit can be 
pushed to the remote for others to pull

# Cherry Picking
Git cherry-pick allows you to incorporate changes from another branch into the current branch:

`git cherry-pick commit_hash1, commit_hash3, ..., commit_hashN`

The above commands appends the selected commits to the commits on the current branch
An interactive rebase allows you to reorder or omit commits from the current branch:

`git rebase -i HEAD~7`

The above command moves you back in the commit tree by seven commits and then allows you to reorder or 
omit commits. You can even change the commit message if you wish
